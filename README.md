# BSDBootImageBuilder

BSDBootImageBuilder is a tool that allows an entire FreeBSD system to compiled
into standalone, monolithic ELF file capable of complete initialization without
any additional support from the runtime environment. To accomplish that, the
builder requires one of more of the initialization modules for the target platform,
a kernel ELF file, a DTB file (if neccessary for the target platform), and,
optionally, a set of kernel modules, including both ELF modules and binary
(e.g., ramdisk) modules.

Currently, the only implemented and tested platform is Xilinx Zynq-7000. See
[BSDKickstart](https://github.com/moon-touched/BSDKickstart) for its
initialization code. Images produced by BSDBootImageBuilder may be directly
linked with Xilinx FSBL through bootgen, without any intermediate bootloaders.

Support for other ARM platforms can be implemented by implementing
the necessary initialization modules. Support for the other architectures
will require significant changes, because the FreeBSD loader ABI
(which we implement) varies significantly between architectures.

# Blueprint files

BSDBootImagesBuilder requires the layout of the output image to be specified
in a form of text-based layout file. An example file, with appropriate
comments, is below.

	; Anything after semicolon is a comment.
    IMAGE_BASE 0x100000 ; IMAGE_BASE specifies the address output image should
						; be linked for. 
    COMPRESS            ; COMPRESS specifies that output image should be 
						; compressed with LZ4 during build and decompressed
						; at startup.

    KICKSTART "BSDKickstart" ; KICKSTART specifies the primary initialization
							 ; module.
							 ;
							 ; Any token can be replaced by a quoted strings.
							 ; Any character, except the newline character, is
							 ; allowed inside quoted string. Quotes and
							 ; backslashes can be escaped by prefixing them
							 ; with a backslash, as usual.
							 ;
							 ; The image must have one and exactly one primary
							 ; initialization module, as this is the first
							 ; piece of code to executed.
    INIT "ZynqInit" ; INIT specifies a secondary initialization module.
					; The image may have any number of the secondary
					; initialization modules, including zero.

	; MODULE specifies a FreeBSD module to be included in the image. The first
	; token after MODULE is a module name, second is a FreeBSD module type
	; (generally, specific values are expected by the kernel), third is the
	; path to the file.
	;
	; METADATA token should be included only for the kernel module and
	; signifies that this module should have kernel metadata generated.
    MODULE kernel "elf kernel" "%kernel%" METADATA
    	DTB DSO100Hardware.dtb ; DTB specifies a DTB (binary device tree) file
							   ; to be passed to the kernel. Optional.
    	KERNEND ; KERNEND signifies that "kernel memory end" value should be
		        ; patched in for this module. Should be always specified.
    	HOWTO 0x840 ; HOWTO specifies kernel boot flags. See the kernel source.
					; Mandatory.
		
		; ENVIRONMENT block specifies the list of sysctl variables to be passed
		; to the kernel. Optional.
    	ENVIRONMENT
    		SET vfs.root.mountfrom cd9660:/dev/md0.uzip
    		SET init_path /DSO100
    	END
    END

	; An example of how an executable module (e.g., a driver) may be specified.
    MODULE dso100fb "elf module" "dso100fb.ko"

	; An example of how a ramdisk module may be specified.
    MODULE rootfs md_image dso100.fs

# Building

BSDBootImageBuilder may be built using normal CMake procedures, and is
generally designed to be run on the host machine in an embedded development
cycle.

# Licensing

BSDBootImageBuilder is licensed under the terms of the MIT license (see LICENSE).