<div align="center">

# ✖ ｌｕａｓｔ－ｄｅｏｂ ✖

### `.:*~*:._ it obfuscates. we unobfuscate. _.:*~*:.`

![python](https://img.shields.io/badge/python-3.10%2B-000000?style=flat-square)
![license](https://img.shields.io/badge/license-MIT-ff0066?style=flat-square)
![tests](https://img.shields.io/badge/tests-14%20passing-000000?style=flat-square)

**[★ JOIN THE DISCORD FOR MORE STUFF LIKE THIS ★](https://discord.gg/9bgECTqeE)**

</div>

---

## `[ x ]` wat is it

static deobfuscator 4 luau scripts wrapped in **luast v1.0.x**. it eats the
garbage and gives u ur code back. no roblox needed. no emulation. just math.

```
$ python3 deobf.py samples/hello.obf.lua --stdout

for i = 1, 3 do
    print(i, "Hello from luast")
end
```

that came out of 160 lines of `aX[137]` and `while true do aj = 12051. - aj`. ♥

## `[ x ]` setup

```bash
unzip luast-deob.zip && cd luast-deob
pip install -r requirements.txt     # optional. it lives without them
python3 deobf.py samples/hello.obf.lua --stdout
```

python 3.10+. nothing 2 build. `numpy` = 100x faster key search,
`zstandard` = unpacks embedded blobs. both optional, both degrade quiet.

## `[ x ]` usage

```bash
python3 deobf.py script.lua                 # -> script.deobf.lua
python3 deobf.py script.lua --report        # did it actually work?
python3 deobf.py script.lua --debug         # pool + state graph, when it dont
python3 deobf.py script.lua --strings --candidates 10
python3 deobf.py script.lua --keep-dead     # keep the anti-tamper junk
python3 deobf.py script.lua --raw           # no renaming / for-loop rebuild
```

read `--report` like this — **states should collapse hard**. 117 in, 4 out = good.
117 in, 117 out = the predicates didnt fold and smthn broke. `0` strings
decrypted or `0` branches folded means same. then u run `--debug`.

## `[ x ]` how it kills

luast keys every string off an **anti-tamper checksum**. it counts
`table.isfrozen(buffer)`, `#debug.info(f,"n")`, font metrics, a `pcall` on a
zstd blob... hook anything, run outside roblox, and every string rots.

so we dont compute it. ✖

the key schedule is an LCG mod 2³², which means byte plane *p* of every round
key only depends on the low `8(p+1)` bits of K₁. so u lift the key one plane at
a time against "plaintext must be printable" — **4 passes of 256 guesses instead
of 4.3 billion**. milliseconds. environment independent. checksum never enters
the chat.

after that its just constant propagation. replay the pool shuffle, fold the
opaque predicates (`x == x`, `0/0 == x`, `v*v < 0`), and 117 states melt into 4.
backward liveness kills the tamper cluster — normal DCE **cant**, those
statements read each other and look alive. then dominators rebuild the loops.

## `[ x ]` it wont save u from

* **names r dead.** luast burns them. u get `v1, v2, i, j`. folded constants get
  inlined. output is *semantically* ur code, not byte-identical.
* **tiny strings.** 4 bytes = mathematically undecidable, we say so instead of
  lying. ~7 bytes = recovered but flagged. ~12+ = basically unique.
* **one sample.** built + verified against one v1.0.1 script. passes key off
  *shape*, and mix/LCG constants are read from the AST not hardcoded, so other
  builds should ride. other versions might not.
* irreducible control flow → `-- [deobf] unstructured jump` comment. it refuses
  to emit wrong code.

## `[ x ]` proof

```bash
python3 tests/test_deob.py      # 14 tests
```

parser round-trip, keystream recovery vs synthetic ciphertext, mix-function
invertibility, and output diffed against the original under a real lua vm.

---

<div align="center">

### `✖ ✖ ✖`

broke it on a new sample? send the script + `--debug` output.
that tells me which pass gave up in about 30 seconds.

### **[★ discord.gg/9bgECTqeE ★](https://discord.gg/9bgECTqeE)**
#### `FOR MORE STUFF LIKE THIS!!`

MIT — see [LICENSE](LICENSE). use it on ur own scripts, audit what ppl tell u to
run, analyze malware. respect what u point it at.

`xXx` *stay dangerous* `xXx`

</div>
