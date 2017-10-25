# boostbuilder

Example usage:

    cmake -H. -Bb -DCMAKE_INSTALL_PREFIX=... \\
        -DBRANCH=boost-1.65.1 -DLIBS="heap;variant"
    cmake --build b --target install
