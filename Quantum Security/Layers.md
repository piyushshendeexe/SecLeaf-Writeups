# Lite

**Category:** Cryptography / Encoding

## Challenge Write-up

The challenge file contained a long string of binary digits. I grouped the bits into 8-bit bytes and converted them to ASCII.

This revealed an alphanumeric string resembling hexadecimal, but the characters `a–f` had been replaced with `n–s`.

I reversed the substitution:

```text
n = a
o = b
p = c
q = d
r = e
s = f
```

After replacing the characters, I decoded the resulting hexadecimal string into bytes.

The output began with `Rar!`, identifying it as a RAR archive. I extracted the archive and recovered the flag.

## Flag

`SecLeaf{F1n4lLy_YOu_G0t_1T_!^!}`
