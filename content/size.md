---
Title: Size Tool
---

Copy your code below, and click submit.

<script>
var keywords = [
    'break',
    'case',
    'catch',
    'class',
    'const',
    'continue',
    'debugger',
    'default',
    'delete',
    'do',
    'else',
    'export',
    'extends',
    'finally',
    'for',
    'function',
    'if',
    'import',
    'in',
    'instanceof',
    'new',
    'return',
    'super',
    'switch',
    'this',
    'throw',
    'try',
    'typeof',
    'var',
    'void',
    'while',
    'with',
    'yield',
    'await',
    'null',
    'true',
    'false',
];

/*
 * Taken from:
 * https://stackoverflow.com/a/18729931/1070117
 */
function toUTF8Array(str) {
    var utf8 = [];
    for (var i=0; i < str.length; i++) {
        var charcode = str.charCodeAt(i);
        if (charcode < 0x80) utf8.push(charcode);
        else if (charcode < 0x800) {
            utf8.push(0xc0 | (charcode >> 6),
                      0x80 | (charcode & 0x3f));
        }
        else if (charcode < 0xd800 || charcode >= 0xe000) {
            utf8.push(0xe0 | (charcode >> 12),
                      0x80 | ((charcode>>6) & 0x3f),
                      0x80 | (charcode & 0x3f));
        }
        // surrogate pair
        else {
            i++;
            // UTF-16 encodes 0x10000-0x10FFFF by
            // subtracting 0x10000 and splitting the
            // 20 bits of 0x0-0xFFFFF into two halves
            charcode = 0x10000 + (((charcode & 0x3ff)<<10)
                      | (str.charCodeAt(i) & 0x3ff));
            utf8.push(0xf0 | (charcode >>18),
                      0x80 | ((charcode>>12) & 0x3f),
                      0x80 | ((charcode>>6) & 0x3f),
                      0x80 | (charcode & 0x3f));
        }
    }
    return utf8;
}

function isSpace(c) {
    return c === 32  /* ' ' */
        || c === 9   /* '\t' */
        || c === 10  /* '\n' */
        || c === 12  /* '\f' */
        || c === 13; /* '\r' */
}

function isLowercase(c) {
    return c > 96 && c < 123;
}

function isCtrl(c) {
    return c === 59   /* ';' */
        || c === 123  /* '{' */
        || c === 125; /* '}' */
}

function checkCode() {
    var elsize = document.getElementById('size-span');
    var elbytes = document.getElementById('bytes-span');

    var code = toUTF8Array(document.getElementById('code-textarea').value);
    console.log(code);

    /* Full size: */
    elsize.innerHTML = code.length + ' bytes';
    if (code.length > 4096) {
        elsize.style.color = 'red';
    } else {
        elsize.style.color = 'green';
    }

    /* Counted bytes: */
    var bytes = 0;
    var word = [];
    var prev = 0;
    var c = code[0];
    if (!isSpace(code[code.length - 1]))
      code.push(10 /* '\n' */);
    for (var i = 0; i < code.length; ++i, prev = c, c = code[i]) {

        /* Currently in a word? */
        if (isLowercase(c)) {
            word.push(c);
            continue;
        }

        /* End of word? */
        if (word.length > 0) {
            /* Word not a keyword. Count it in full! */
            var w = word.map(function(e){return String.fromCharCode(e);}).join('');
            if (!keywords.find(function(e){ return e === w; })) {
                bytes += word.length;
                word = [];
                continue;
            } else {
                bytes += 1;
                word = [];
            }
        }

        /* Ignore any spaces. */
        if (isSpace(c)) {
            /* Ignore curly braces and semicolons if followed by whitespace. */
            if (isCtrl(prev)) {
                bytes -= 1;
                continue;
            }

            continue;
        }

        /* Count the byte as normal. */
        bytes++;
    }

    elbytes.innerHTML = bytes.toString();
    if (bytes > 2048) {
        elbytes.style.color = 'red';
    } else {
        elbytes.style.color = 'green';
    }
}

</script>

<form onsubmit="return false">
    <textarea id="code-textarea" name="code" cols="80" rows="20"></textarea>
    <br>
    <input type="submit" value="Update" onclick="checkCode()">
</form>

**Full size:** <span id="size-span"></span><br>
**Counted bytes:** <span id="bytes-span"></span>

The full size must not exceed 4096 bytes (rule §2.1).<br>
The number of counted bytes must not exceed 2048 (rule §2.2).
