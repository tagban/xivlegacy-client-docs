# Architecture

This page describes the approach and what we learned. It doesn't contain the code or the steps to
rebuild it.

## The executable

`ffxivgame.exe` 2012.09.19.0001 is a 32-bit x86 Windows program built with Visual Studio 2005:
- about 16 MB, with 11.8 MB of code (roughly twice FINAL FANTASY XI's);
- image base 0x400000, no relocations, not packed;
- a TLS directory and a leftover `MSSMIXER` section from the Miles sound library.

The disc doesn't contain it: the first game patch adds it.

## Static recompilation

Every function is found ahead of time, and the x86, x87, SSE/SSE2 and MMX instructions are
translated to C, which is then compiled natively. The approach is the same as our FINAL FANTASY XI
native client. A few things are specific to this binary:

- **Hand-written jump tables:** the C runtime's x87 math dispatch jumps through unaligned tables
  that function discovery has to be taught about.
- **Code outside `.text`:** the `MSSMIXER` section holds a CPU feature check, which gets a native
  stand-in.
- **Flags must be stored:** returning without storing CPU flags crashed at the title screen. Some
  code reads flags that a callee left behind.
- **The clock:** the lobby refuses to work unless the clock reads near the end of 2012, the way
  the community's 1.x clients also patch it.

## The portable runtime

The game sees a small Windows built for it:
- kernel32, user32 (windows and input through SDL3), ws2_32, ole32, the registry;
- a mounted file system, fibers, WinHTTP;
- DirectInput/XInput, XAudio2 2.4 (on SDL3 audio), Direct3D 9 and D3DX.

Each API does what this game needs, not everything Windows does.

## Threads and the guest lock

The translated code isn't thread-safe against itself, so all guest threads share one lock.

- **Long turns for the drawing thread:** 8 ms turns held the world at 26 fps, 50 ms turns at 40.
  The drawing thread gives the lock up whenever its frame sleeps.
- **The drawer gets right of way:** only the thread that draws (identified when it first presents)
  gets the long turn and the urgent queue position. Other threads used to grab long turns too,
  which caused walking stutter.
- **Audio refills in batches:** the XAudio2 engine fills its device queue in 50-80 ms batches when
  the lock comes free, and asks for it urgently only when the queue runs low.

## The launcher

A small Swift app:
- signs in through the server's web login and remembers the account in the Keychain;
- starts the local server when it's on localhost and not answering;
- runs the game as a helper process and reappears when the game exits.

Every local build is signed with a Developer ID, so the Keychain's "Always Allow" survives
rebuilds.
