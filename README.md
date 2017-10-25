# boostbuilder

Example usage:

    cmake -H. -Bb -DCMAKE_INSTALL_PREFIX=... \\
        -DBRANCH=boost-1.65.1 -DLIBS=heap
    cmake --build b --target install
