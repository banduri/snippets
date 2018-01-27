Installation
============

Install packages on Archlinux
 pacman -S extra/linux413-nvidia extra/linux413-nvidia-340xx extra/nvidia-utils extra/opencl-nvidia

Active 340xx driver 
 mhwd -i video-nvidia-340xx pci


Minersoftware
=============

Ethereum:

 git clone https://github.com/ethereum-mining/ethminer
 cd ethminer
 mkdir build
 cd build
 cmake -DCMAKE_C_COMPILER=/usr/bin/gcc-6 -DCMAKE_CXX_COMPILER=/usr/bin/g++-6 -DETHASHCUDA=ON -DCUDA_TOOLKIT_ROOT_DIR=/opt/cuda/ ..

 cmake --build .


Minerpools
==========

 https://ethereumpool.co/how/

 ./ethminer -G -F http://ethereumpool.co/?miner=25.60@0x486CA34516E53270b1AFc6346b796C4e79b74bd7@taz
