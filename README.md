lua_base64
==========

Lua base64 encoding and decoding (Lua 5.2)

I was looking for some "native to Lua" base64 encoding and decoding routines
and didn't find any that were "fast enough." With Lua 5.2 and the new bit32
routines I suspected that a better job could be done. The best routine I found
really used a bunch of memory. For an 800K file, over 6.5M was needed for an
intermediate value. For my use, this was way too much. The alternative of
doing a popen("base64") was considered, but isn't overly portable.  Speed was
an issue when 800K files took over 2 full seconds.

This module "exports" 5 methods with various "overloads" that allow
interaction with the encoding / decoding routines. Default is to encode and
decode as RFC 2045 (Ignores max line length restrictions). The input and
output in this mode is generally compatible with the GNU base64 application.

**Tested against base64 (GNU coreutils) 8.22 & 8.13.**


###Basic Usage

The simplest is "string in" / "string out".

```lua
base64=require("base64")

print( base64.encode("This is a string") )
print( base64.decode("RHVkZSEgV2hlcmUgaXMgbXkgY2FyPz8/Cg==") )

--[[ Output

VGhpcyBpcyBhIHN0cmluZw==
Dude! Where is my car???

]]--
```

For "very large strings" this ~~may not be~~ is not the best use of the
library.
> In fact, I likely wouldn't encourage using these routines **at all**
for large strings. A Lua c-module will handle this _considerably_ faster. Just
"for fun" I ran a 600M file through base64 and this utility. 0m3.306s vs
8m13.139s. It is plausible that base64 spun multiple threads, but I suspect
that the real reason is that all of the "over head" of a function call vs an
extremely tight and optimized C routine is the real factor. Granted, this is a
perverse example.


###More Examples:

####stdio
```lua
base64=require("base64")
ii=base64.encode_ii(io.stdin)
base64.encode(ii,function(s) io.write(s) end)

--[[ Output

$lua test.lua < base64.lua

LS1bWyoqKioqKioqKio ... dGVyYXRvcgp9Cg==

]]--
```

#### output predicate
```lua
o={}
base64.encode("Encode this please",function(s) o[#o+1]=s end)
for i,v in ipairs(o) do
    print(i,v)
end

--[[ Output

1   RW5j
2   b2Rl
3   IHRo
4   aXMg
5   cGxl
6   YXNl

]]--
```

#### output predicate
```lua
function linespliter()
    local c = 0
    return function(s)
        io.write(s)
        c=c+1
        if c > 5 then
            io.write("\n")
            c=0
        end
    end
end

f=io.open("base64.lua")
s=f:read("*a")
f:close()
base64.encode(s,linespliter())

--[[ Output

LS1bWyoqKioqKioqKioqKioq
KioqKioqKioqKioqKioqKioq
        . . .
ZGVjb2RlX2lpICAgPSBkZWNv
ZGU2NF9pb19pdGVyYXRvcgp9
Cg==

]]--
```


#### garbled input
```lua
-- Mess with the input "V2hhdCBpcyB0aGlzPwo="
s="V 2 h h d C Bp c y(((@!!!!\n\n\r\t\tB0aGlzPwo=           :-)     ?"
print(base64.decode(s))

--[[ Output

What is this?

]]--
```


####RFC 4648 'base64url'
```lua
base64.alpha("base64url")
i=io.open("foo")
o=io.open("bar","w")
o:write(base64.encode(i:read("*a")))
i:close()
o:close()

--[[ No output ]]--
```

####custom alphabet
```lua
base64.alpha("~`!1@2#3$4%\t6^7&8*9(0)_-+={[}]|\\:;'<D,./?qwertyuioplkjhgfdsazxcv","")
s=base64.encode("User base64 encoding, no term chars")
print(s)
print(base64.decode(s))

--[[ Output

)-^,}'`'+-^,^<8:=_d<[h*q[.}r$#du$3*,}.k:+h;;}/6
User base64 encoding, no term chars

]]--
```
