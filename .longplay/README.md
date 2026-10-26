This is Longplay's fork of [SDL](https://github.com/libsdl-org/SDL): an
upstream release with a few patches on top (below), and the files that make
the repository a Cargo package, `sdl3-src`, which `sdl3-sys` builds SDL from.

## Using it

Each `lp/<version>` branch is SDL `release-<version>` plus our commits. In a
Rust workspace:

```toml
[patch.crates-io]
sdl3-src = { git = "https://github.com/longplay-app/SDL", branch = "lp/3.4.16" }
```

The version has to be the one `sdl3-sys` wants (`cargo tree -i sdl3-src`; for
`sdl3-sys 0.7.1+SDL-3.4.16` it is `=3.4.16`). Cargo.lock pins the commit.
Without Rust, it is SDL: build it with CMake as usual.

## What's different from upstream

Everything after the release tag: `git log release-3.4.16..lp/3.4.16`. The
files that only exist here are `Cargo.toml` and `.longplay/`.

- **Transparent windows for the GPU API**, so a window can be borderless with
  rounded corners (and Direct3D 12 can present one, through DirectComposition).
  Vulkan needed only a check relaxed.
- A GPU window destroyed while still claimed is released.
- Direct3D 12 can fall back to WARP (`SDL_HINT_GPU_D3D12_WARP`).

## Updating to a new SDL release

    git fetch upstream --tags            # upstream = libsdl-org/SDL
    .longplay/update 3.4.18

That makes `lp/3.4.18` by rebasing the last branch's commits onto
`release-3.4.18`; git stops at a commit that no longer applies, and you fix it
as in any rebase (`git rebase --continue`), then `git push origin lp/3.4.18`.
The script also sets the version in `Cargo.toml` and `.longplay/lib.rs`, in a
commit of its own at the end. Then point the workspace at the new branch and
move `sdl3-sys` to the version for that release.

To change a patch, check out the branch, edit, `git commit --fixup=<commit>`
and `git rebase -i --autosquash release-<version>`; or just add a commit.
Keep patches that could go upstream apart from ones that can't, and offer the
former to SDL.
