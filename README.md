# Installing ZoKrates on Mac with Libsnark Enabled

I recently started to learn how to use ZoKrates. The default installation only supports the proving scheme G16, which is malleable, that is, an attacker can easily generate another valid proof if he/she sees a valid proof. 

It took me quite some time to figure out how to properly install ZoKrates on my Mac to enable the other two proving schemes and I feel like it is worth sharing with others.

## Installation Step

1. Download ZoKrates using `git clone https://github.com/Zokrates/ZoKrates.git`

2. Modify `$zokrates_home/zokrates_core/build.rs` as follows:
	
	```rust
	let libsnark = cmake::Config::new(libsnark_source_path)
            .define("WITH_SUPERCOP", "OFF")
            .define("WITH_PROCPS", "OFF")
            .define("CURVE", "ALT_BN128")
            .define("USE_PT_COMPRESSION", "OFF")
            .define("MONTGOMERY_OUTPUT", "ON")
            .define("BINARY_OUTPUT", "ON")
            .build();       
	```
	
3. Make sure that `openssl` has been installed. Otherwise, install via `brew install openssl`.

4. Set `export PKG_CONFIG_PATH=$(brew --prefix openssl)/lib/pkgconfig`

5. Set `export WITH_LIBSNARK=1`

6. Run `build_release.sh` and the compiled executable `zokrates` can be found in `$zokrates_home/target/release/`

## A Simple Example

As a simple example, I want to prove my knowledge of two secret numbers whose sum equals 10. To do that using ZoKrates, we first write file `simple_add.code` as:

```
def main(private field a, private field b) -> (field):
    a+b == 10
    return 1
```

Suppose we want to use proving scheme `GM17` and the secret numbers are 3 and 7, we do:

```
zokrates setup -s gm17
zokrates export-verifier -s gm17
zokrates compute-witness -a 3 7 
zokrates generate-proof -s gm17
```

Within the current folder, we can see the exported contract `verifier.sol` that implements the verification function and the proof `proof.json`. 

