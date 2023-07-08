V now compiles itself in a stunning 0.09 seconds:


```bash
wget https://github.com/vlang/v/releases/download/0.1.21/v_linux.zip
unzip v_linux.zip && cd v
./v -o ./v2 v.v # warm up
time ./v -fast -o ./v2 v.v
0.06s user 0.03s system 97% cpu 0.094 total
./v2 -o v3 v.v # make sure that V can still build itself
```

My desktop is pretty fast, let's try a slower machine. On a 5 year old laptop with a mobile CPU, I'm getting 0.31 seconds, which is still very fast.

This was achieved by several optimizations, like caching the `builtin` module and direct machine code generation via tcc. 


There are still a couple of things to optimize. By January 2020 it should drop to 0.06 seconds.

Everything has a price though: this is only for unoptimized builds, which are ~2 times slower than production builds (this is still fast enough for a vast majority of users during the development cycle). Running `v -prod` to make a production build is going to be significantly slower.

### Easier bootstrapping without dependencies

Another cool change is an even simpler bootstrapping process. Bootstrapping V has always been simple from the start:

```
cc -o v v.c
```

No need to install any dependencies, libraries, etc. Just building a single C file with a C compiler (by the way, soon it won't even have any includes for better portability).

However, a C compiler was still needed. It's small and easy to install dependency that's very likely to already be installed on your machine, but it's still a dependency.

From now on the 2.6 MB zip archive contains everything you need to build V:

```bash
wget https://github.com/vlang/v/releases/download/0.1.21/v_linux.zip
du -h v_linux.zip
2.6M v_linux.zip
```

This has only been set up for Linux for now. It'll work on Windows by the end of the week. macOS users will have to get used to the "slow" 0.4 - 1.2 second builds until January.
