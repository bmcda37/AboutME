
/* Demonstrate the binary loader from ../inc/loader.cc */

#include <stdio.h>
#include <stdint.h>
#include <string>

#include "../inc/loader.h"

void dump_section_contents(const Section &sec) {
    printf("\nDumping contents of section: %s\n", sec.name.c_str());
    for (size_t i = 0; i < sec.size; i++) {
        if (i % 16 == 0) {
            printf("\n0x%016jx: ", sec.vma + i);
        }
        printf("%02x ", sec.bytes[i]);
    }
    printf("\n");
}

int main(int argc, char *argv[]) {
    if (argc < 3) {
        printf("Usage: %s <binary> <section_name>\n", argv[0]);
        return 1;
    }

    std::string fname = argv[1];
    std::string section_name = argv[2];
    Binary bin;

    if (load_binary(fname, &bin, Binary::BIN_TYPE_AUTO) < 0) {
        return 1;
    }

    printf("loaded binary '%s' %s/%s (%u bits) entry@0x%016jx\n", 
           bin.filename.c_str(), 
           bin.type_str.c_str(), bin.arch_str.c_str(), 
           bin.bits, bin.entry);

    Section *sec = nullptr;
    for (size_t i = 0; i < bin.sections.size(); i++) {
        if (bin.sections[i].name == section_name) {
            sec = &bin.sections[i];
            break;
        }
    }

    if (sec) {
        dump_section_contents(*sec);
    } else {
        printf("Error: Section '%s' not found in the binary.\n", section_name.c_str());
    }

    unload_binary(&bin);
    return 0;
}
