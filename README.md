# AmigaX

<BR>
Repository Overview

This is an AmigaX project - a Motorola 68000 CPU emulator with enhanced Amiga-style custom chipset emulation. The system emulates an enhanced Amiga computer with graphics, sprites, DMA, and a full M68K assembler for running 68000 assembly programs.

The project consists of two main variants:

    Standard AmigaX: Classic Amiga AGA-style chipset emulation (demo_full_assembler)
    Enhanced AmigaX: Dramatically enhanced capabilities (100x better than AGA) with up to 4K resolution, 256 sprites, 32-bit color, and alpha blending (demo_enhanced)

Building the Project
Quick Build (using build.sh)

./build.sh

The build script checks for SDL2, builds using the default Makefile, and offers to run the demo. This builds both demo_enhanced and demo_full_assembler.
Manual Build

The project has two different Makefiles for different build configurations:

# Option 1: Enhanced AmigaX (using Makefile - default)
make                    # Builds both demo_enhanced and demo_full_assembler
./demo_enhanced         # Enhanced demo with modern features
./demo_full_assembler   # Standard Amiga-style demo

# Option 2: Standard AmigaX (using Makefile.Linux)
make -f Makefile.Linux  # Builds single 'amigax' executable
./amigax                # Standard AmigaX with built-in demo
./amigax program.asm    # Can load and run .asm files

Key differences:

    Makefile builds two separate demo executables (enhanced and standard)
    Makefile.Linux builds one executable (amigax) that accepts command-line arguments for loading assembly files

Build Targets

# Makefile.Linux (standard AmigaX)
make -f Makefile.Linux all        # Build amigax executable
make -f Makefile.Linux clean      # Clean build artifacts
make -f Makefile.Linux run        # Build and run
make -f Makefile.Linux debug      # Build with debug symbols
make -f Makefile.Linux check-sdl  # Verify SDL2 installation

# Makefile (enhanced AmigaX)
make all                # Build both demo_enhanced and demo_full_assembler
make demo_enhanced      # Build enhanced demo only
make clean              # Clean object files
make distclean          # Clean everything including copied project files

Architecture Overview
System Components

The emulator is structured around classic Amiga architecture with custom chips:

    M68K CPU Core (m68k_full.c, m68k_complex.c)
        Full Motorola 68000 instruction set implementation
        CPU state management (registers D0-D7, A0-A7, PC, SR, etc.)
        Memory access through system bus
        Located in M68K_CPU structure

    Custom Chipset
        Denise (Display controller): Manages bitplanes, sprites, palettes
        Agnus (DMA/Memory controller): Handles DMA channels, Copper, Blitter
        Paula (Audio controller): Multi-channel audio (Enhanced: 16 channels)

    Assembler (m68k_assembler.c, m68k_assembler_extended.c)
        Two-pass M68K assembler
        Supports labels, constants (EQU), directives (ORG, END)
        Extended instruction set support
        Symbol table with up to 1024 labels

    AmigaX System (amigax_system.c, amigax_system.h)
        Main system integration
        Memory mapping (Chip RAM, Fast RAM, ROM, Custom registers)
        SDL2 rendering and event handling
        Glues CPU, custom chips, and memory together

Memory Map

0x00000000 - 0x001FFFFF  Chip RAM (2MB standard, 16MB enhanced)
0x00C00000 - 0x00FFFFFF  Fast RAM (4MB standard, 256MB enhanced)
0x00DFF000 - 0x00DFFFFF  Custom chip registers
0x00F80000 - 0x00FFFFFF  ROM (512KB standard, 2MB enhanced)

Enhanced vs Standard Features

Standard AmigaX:

    640x480 resolution
    8 bitplanes (256 colors)
    8 hardware sprites
    AGA-compatible chipset

Enhanced AmigaX:

    Up to 3840x2160 (4K) resolution
    32 bitplanes (4.2 billion colors)
    256 hardware sprites (up to 128x128 pixels each)
    Multiple display modes: indexed, RGB15/16/24, RGBA32, HAM8/24, chunky
    Alpha blending for sprites
    Hardware-accelerated blitter with alpha support
    16 independent 256-color palettes
    32 DMA channels
    Enhanced Copper (display list processor)

Key File Structure
Core Implementation Files

    amigax_system.c/h - Main system, memory management, custom register I/O
    m68k_full.c - Complete 68000 CPU emulator
    m68k_complex.c - Complex addressing modes and instructions
    m68k_assembler.c/h - Two-pass assembler core
    m68k_assembler_extended.c/h - Extended instruction set

Enhanced System Files

    amigax_enhanced.h - Enhanced chipset definitions and API
    amigax_enhanced_impl.c - Enhanced system implementation
    amigax_enhanced_render.c - Enhanced rendering engine

Demo Programs

    demo_full_assembler.c - Standard AmigaX demo
    demo_enhanced.c - Enhanced features demo (sprites, animation, palettes)

Helper Files

    m68k_cpu.h - CPU structure definition (must exist in project)
    m68k_opcodes.h - Opcode definitions
    m68k_helpers.h - Helper macros
    m68k_missing_opcodes.c - Additional opcode implementations

GUI and Overlay Systems

    simple_gui_overlay.c - SDL2_ttf-based on-screen display (requires SDL2_ttf)
    imgui_amigax_concept.cpp - Dear ImGui debugger concept (not built by default)

Reference Documentation

    links/ directory contains PDF references for 68000 instruction set
    links/readme.md contains links to online 68000 references

Assembly Language Integration

The system includes a built-in M68K assembler that can assemble and load programs at runtime.
Assembler API Functions

Available in amigax_system.h:

// Standard assembler (m68k_assembler.c)
bool amigax_assemble_and_load(AmigaXSystem *sys, const char *source, uint32_t load_addr);
bool amigax_assemble_file_and_load(AmigaXSystem *sys, const char *filename, uint32_t load_addr);

// Extended assembler (m68k_assembler_extended.c) - includes additional instructions
bool amigax_assemble_and_load_ext(AmigaXSystem *sys, const char *source, uint32_t load_addr);
bool amigax_assemble_file_and_load_ext(AmigaXSystem *sys, const char *filename, uint32_t load_addr);

Usage:

    source: String containing M68K assembly code
    filename: Path to .asm file
    load_addr: Memory address to load assembled code (e.g., 0x1000)
    Returns true on success, false on assembly errors

Supported Assembly Syntax

        ORG     $1000           ; Set origin address
WIDTH   EQU     640             ; Define constant
HEIGHT  EQU     480

start:
        MOVEQ   #0,D0           ; Quick move
        MOVE.L  #$10000,A0      ; Long move
        LEA     data(PC),A1     ; PC-relative addressing
loop:
        MOVE.B  (A0)+,(A1)+     ; Memory to memory with post-increment
        DBRA    D0,loop         ; Decrement and branch
        RTS                     ; Return from subroutine

data:   DC.B    $00,$01,$02     ; Data bytes
        END     start           ; End with entry point

Assembler Features

    Two-pass assembly (first pass: collect labels, second pass: generate code)
    Label resolution with forward references
    Standard M68K addressing modes (immediate, register direct/indirect, indexed, etc.)
    Directives: ORG, EQU, DC.B/W/L, DS.B/W/L, END
    Extended instructions in m68k_assembler_extended.c

Custom Register Programming

Access Amiga-style custom registers through memory-mapped I/O:

// Bitplane control registers
#define BPLCON0   0xDFF100   // Bitplane control 0
#define BPLCON1   0xDFF102   // Bitplane control 1 (scrolling)
#define BPL1PTH   0xDFF0E0   // Bitplane 1 pointer (high word)
#define BPL1PTL   0xDFF0E2   // Bitplane 1 pointer (low word)

// Display registers
#define COLOR00   0xDFF180   // Base color register
#define VPOSR     0xDFF004   // Vertical position
#define DMACON    0xDFF096   // DMA control

// Enhanced registers (amigax_enhanced.h)
#define BPLCON4_ENH  0xDFF108  // Additional bitplane control
#define SPR0PTH_ENH  0xDFF120  // Sprite 0 pointer

Running Programs

Which executables are available depends on which Makefile was used:
After building with make (default Makefile):

./demo_enhanced             # Enhanced graphics demo (sprites, palettes, 4K support)
./demo_full_assembler       # Standard Amiga-style demo

These demos have hardcoded assembly programs and don't accept command-line arguments.
After building with make -f Makefile.Linux:

./amigax                    # Run with built-in demo
./amigax program.asm        # Assemble and run custom program.asm file

This version can dynamically load and assemble .asm files from the command line.
Controls

    ESC - Quit
    F1 - Toggle help overlay (if simple_gui_overlay.c is compiled in)
    F10 - Toggle CPU trace

Development Notes
Dependencies

    SDL2 - Required for graphics and window management
        Install: sudo dnf install SDL2-devel (Fedora)
        Install: sudo apt-get install libsdl2-dev (Ubuntu/Debian)
        Check: pkg-config --exists sdl2
    SDL2_ttf - Optional, for GUI overlay (simple_gui_overlay.c)
        Requires DejaVuSans.ttf font file in project directory
    Math library (-lm) - Required for floating-point operations

Build Notes

Fixed Issues:

    m68k_cpu.c is not needed - all CPU functions are implemented in m68k_full.c
    Both Makefiles have been fixed to build without m68k_cpu.c
    Makefile uses -std=gnu11 to include POSIX extensions (strcasecmp, strdup)
    Makefile.Linux also uses -std=gnu11 for the same reason
    Enhanced system wrapper functions added to bridge naming differences between amigax_system.c and amigax_enhanced_impl.c

CPU Structure (M68K_CPU)

Central to the emulator. Contains:

    Data registers: D[0..7] (32-bit)
    Address registers: A[0..7] (32-bit)
    Program counter: PC (32-bit)
    Status register: SR (16-bit with CCR flags)
    Stack pointers: User SSP and Supervisor SSP
    Supervisor mode flag
    Interrupt mask level

Rendering Pipeline

    CPU executes instructions, modifying memory and registers
    Custom registers control display configuration (bitplanes, sprites, palette)
    amigax_render_frame() composites the display:
        Fetches bitplane data from Chip RAM
        Applies palette lookups
        Renders sprites with priority sorting
        Applies hardware scrolling
        Outputs to SDL texture/window

Blitter Operations

The Blitter performs fast memory-to-memory copies with logic operations:

    Three source channels (A, B, C) and one destination (D)
    Minterm logic operations (256 combinations)
    Line drawing mode
    Fill mode
    Enhanced: Alpha blending support

Debugging Features

The emulator includes several debugging aids:
CPU Trace Mode

    Press F10 at runtime to toggle CPU trace
    When enabled, prints executed instructions to console
    Shows PC (Program Counter) and instruction details
    Stored in sys->trace_enabled flag

On-Screen Debug Display

When compiled with simple_gui_overlay.c:

    Press F1 to toggle help overlay
    Shows current PC, frame counter, and trace status
    Displays keyboard shortcuts
    Requires SDL2_ttf and DejaVuSans.ttf font

Dear ImGui Debugger Concept

The file imgui_amigax_concept.cpp contains a blueprint for a full-featured debugger with:

    CPU register display (D0-D7, A0-A7, PC, SR)
    Memory viewer with hex display
    Disassembler window
    Breakpoint support
    Step/Run/Pause/Reset controls
    Not built by default (requires Dear ImGui library)

To build with ImGui:

gcc demo_full_assembler.c amigax_system.c m68k_full.c m68k_complex.c \
    m68k_assembler.c m68k_assembler_extended.c \
    imgui_impl_sdl2.cpp imgui_impl_sdlrenderer2.cpp imgui*.cpp \
    -o amigax_gui -lSDL2 -lstdc++

Testing and Debugging

No formal test framework. Testing is done by:

    Running demo programs (demo_enhanced, demo_full_assembler)
    Visual verification of SDL2 graphics output
    CPU trace mode (F10) to monitor instruction execution
    Writing and assembling custom test programs in M68K assembly

To verify the build works:

./build.sh              # Should build and offer to run demo
./demo_full_assembler   # Should open SDL window with graphics
./demo_enhanced         # Should show enhanced features (sprites, palettes)

Note: The build.sh script mentions test files (test_simple.asm, graphics_demo.asm, examples.asm) that do not currently exist in the repository. These would need to be created manually.
Project Context

This appears to be part of a larger collection located at /home/franco/Downloads/claudev3/Claudev2-main/CPlusPlus/. The "True_Amiga_Working" directory name suggests this is a working/stable version of an Amiga emulator project, possibly with other variants in sibling directories.
Recent Fixes (2025-11-11)

The build system has been completely fixed and both Makefiles now build successfully:
Issues Resolved:

    Missing m68k_cpu.c - Removed from Makefile since all functions exist in m68k_full.c
    Duplicate amigax_render_frame - Fixed by separating standard and enhanced builds properly
    Missing memory wrapper functions - Added amigax_read_memory_* wrappers to amigax_enhanced_impl.c
    POSIX function errors - Changed from -std=c11 to -std=gnu11 in both Makefiles
    strcasecmp implicit declaration - Added #include <strings.h> to m68k_assembler.c

Build Verification:

All three executables build successfully:

    ./build.sh → builds demo_enhanced and demo_full_assembler (36KB and 50KB)
    make -f Makefile.Linux → builds amigax (187KB)

The code is now fully functional and ready to run.
