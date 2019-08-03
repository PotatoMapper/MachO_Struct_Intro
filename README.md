# MachO_Struct_Intro #
### A Short and Sweet Document on MachO executable Structure and MacOS/iOS dylad_shared_cache Written with a Target Audience expected to have basic knowledge of binary executables:

## Basics:

**MacOS/iOS programs (Like any other OS) rely on Libraries**

Libraries contain shared code used by different programs

When Program is initialized it Loads all the libraries it needs and links all the libraries to allow them to call each other's functions.

In addition, the ObjC runtime must Init each library when its loaded.

*These tasks inherently slow down Launching of programs and in comes Apple ingenuity*

*macOS and iOS improve startup time and Mem Usage by combining ALL system Libraries*
into the **dyld shared cache**

**dyld shared cache:** 
File containing EVERY library built into the OS, all linked together and with the ObjC runtime already initialized. So instead of loading and processing Hundreds of Files on file launch, the program just loads one file at startup already Pre Processed.

**On iOS, since ALL the system libraries are in the shared cache, the individual Libraries
are removed to save space.**

---
### What If I Want To Modify or Extract A Single Core Library for my own modification?

*It is possible to load an iOS system Library Seperately, but to get the Library, you must
extract it from the Shared Cache, and consequently means undoing all the preprocessing when 
building the shared cache*

*Currently Available Tools for Extracting the dyld_shared_cache*

Each of these solutions have their shortcomings:

[jtool](http://www.newosxbook.com/tools/jtool.html): Frequently updated and maintained but doesn't fix the ObjC Selectors

[decache](https://github.com/phoenix3200/decache): Deprecated, last functioned on iOS 9, pre 64bit iOS Devices

Apples own [dsc_extractor](https://opensource.apple.com/source/dyld/dyld-519.2.2/launch-cache/dsc_extractor.cpp.auto.html): Used in Xcode when 1st plugging in a device for Debugging
(The "Preparing Debugger Support" prompt is this actually) Libraries are only usable for providing symbols to debuggers this way

[imoan2](https://github.com/comex/imaon2): The wiki says it produces the Highest Quality Output, but dependencies make it a bitch and a half to compile [Supports iOS 11]

Resource on Retrieving Librarie from dyld_shared_cache available @ [WorthDoingBadly.com](https://worthdoingbadly.com/dscextract/)

---
## ELI5 Reference for MachO Compiled Binary Structure

**Header Structure**
Comprised of series of Load Commands

Tells dyld (macOS Dynamic Library Loader) info about File

Some Load specify Metadata about File

      ex) Target OS Version, File Entry Point

**MOST IMPORTANT ARE THE LOAD COMMANDS THAT DEFINE SEGMENTS**

Segments are parts of MachO file that are loaded into memory @ SPECIFIED ADDRESS

**Main 3 Segments in most Libraries**

      "__TEXT": Holds Code and Data that Does Not change 

      "__DATA": Holds Data that DOES change 

      "__LINKEDIT": Holds instruction for the dyld to:
      1) Relocate the library to correct Mem ADDRESS
      2) Import Functions it Needs
      3) Export Functions it Contains
      
Each Segment can also define **SECTIONS**

**Sections** further Subdivide the segment into named parts

      *EX) __DATA includes Section such as __objc_cfstring, containing NSStrings, and __objc_classlist, Containing Pointers to ObjC classes defined in the file*

(The Concept of Segments and Sections also exist in *ELF*, the executable format used by Linux and other Modern Unix Systems)
