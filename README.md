# TADA

(work in progress)

"Totally Awesome Dungeon Adventure" (TADA) is a Commodore 64 re-implementation of the Apple BBS game "The Land of Spur" (TLOS). But instead of being a single-player, one-at-a-time, multi-user-dungeon as it was in the dial-up BBS days, I would like to eventually leverage the server-client technology of [CommodoreServer](https://www.commodoreserver.com) to have a real, multi-player game experience.

TLOS was written in a scripting language called _Advanced Communications Operating System_ (ACOS). It had limitations and quirks, as any programming language does. One such quirk is in handling signed integers between -32767 and +32767. Naturally, this is a bit restrictive when dealing with adventure game statistics such as the amount of gold carried upon your person, or similar things. (There are cumbersome workarounds in the code for this, splitting larger values into "high and low" halves.) In my rewrite, I address this -- with routines written by FuzzyFox of "AutoGraph," a graphics converter for the C64, fame -- with essentially 24-bit arithmetic, yielding values of 1 to 1,677,214. -- a much more comfortable range.

Another annoyance, at least to me, was that there was a lot of duplicate, shared code between modules. This rewrite addresses that by having a "kernel" in memory at all times with common subroutines, callable by any module when loaded into RAM from disk.

## Advantages of C64 framework:
* Common kernel routines: code isn't repeated throughout each module as is in TLOS.
* Modular: More, smaller modules make it hopefully easier to write/upgrade routines.
* [modBASIC](https://www.commodoreserver.com/BlogEntryView.asp?EID=EB7662805E4B4A7ABA2623257BCC642E): Parameter-passing and local variables unlike the laundry list of obscure variables used in TLOS, not all of which I have documented yet.
* [C64List](http://commodoreserver.com/BlogView.asp?BID=620460DB83BF4CC1AE7FEF4E9AB4A228): Written in an easy-to-read, friendly text format which can be translated to C64 BASIC, or assembled to 6510 assembly code
* Similar routine labels to TLOS source

## Directory structure:
`SPUR-data`: Data files for study, part of TLOS. 
`SPUR-code`: Modules for TLOS.
`assembly-language`: C64 6510 assembly projects: $c500 code, parser, works in progress, or code testing.
`programming-notes`: Notes on both TLOS and TADA.
`scripts`: Build, automation, and test scripts for both M$-DO$ and Linux.
`text-listings`: This is the meat of the archive, with C64 modules of various types for TADA. Inside this directory:
  `editors`: Some notable things:
    * `tep82.lbl`: Character editor (mostly for fixing bad values written during the "new player" routine). Later, once this becomes modularized, players with _Dungeon Master_ status will be allowed to use this.
    * `ltk-editor.lbl`: Possibly going to be abandoned in favor of storing strings under ROM, and using a fancier input routine, once that is debugged.
    * `teo.lbl`: TADA Object Editor. This file format will possibly be abandoned, but it has the bones of a good idea. If items have been destroyed and need to be re-introduced to the game, or if the value of a treasure needs to be adjusted, a `Dungeon Master` would conceivable use this program to do so.
  `includes`: files used by other files. Saves typing. All the modern programmers do it.
  `installers`: Not really traditional "installers" in the sense of copying files from an archive to directories, they do nevertheless change SEQuential files into RELative ones, usually.
  `misc`: Tests, weird one-off "I wonder how this works" ideas, unfinished stuff, unrealized features, maybe even from other future projects.
  `tests`: Tests of routines, hopefully not littering source code proper, but there's probably some of that, too.
`text`: Text captures from TLOS, miscellaneous notes.

## Assembly language routines:
* Module Load: Takes code from "Module 64" from a Compute! article. This setup uses a main BASIC program ("kernel" in my terminology) and is always resident in RAM. The kernel uses lines 1000-. Loadable modules use lines 1-999, "linked" together with the kernel at load time. BASIC variables are preserved between loads (this requires the largest loaded module to be combined with the "kernel" when the kernel is started, which sets top-of-BASIC pointers to save the start-of-variables pointer). Modules can call any subroutine within the kernel ending in a RETURN.
* Input Any: A routine which handles disk file input. Traditionally, e.g. `INPUT #2,x$` will truncate lines of disk file data containing commas or colons, and can't handle binary data (such as my player log file). A SYS call returns all data into a string variable, getting up to a specified number of bytes, either stopping at or ignoring a carriage return.
* Bracket Text: Display a string such as "\[Hello] there" with "Hello" highlighted in a different output color.
* PopStack: Pops `return` addresses off the stack. There are a lot of uses of this in The Land of Spur, unfortunately.
* InString: Replicates `INSTR("search_through","search_for")` function in ACOS, or other BASICs, such as the C128.
* String Array System: By Jeff Jones. This allows strings to be stored beneath ROM. Hopefully it will save some BASIC RAM. I haven't implemented much other than a demo of it being used yet, however.
* Sliding Input: Originally by Creative Micro Designs. With help, I have enhanced this utility to integrate with BASIC. It passes a string from the SYS call to be edited, and unlike `INPUT`, traps against accidental Clr/Home keypresses (confirmation is provided if you really _do_ want to erase the string), allows Inst/Del usage (while defeating "quote mode"), and `f1` and `f7` move left and right by words. (If the allowed length of the input exceeds the width of the "window" for displaying the input, the input scrolls left or right, hence the name "Sliding Input.") Hit `Return` and the string is passed back to BASIC, replacing the original contents.

## Want to play the original game?
It's telnettable: telnet://dura-bbs.net:6359
