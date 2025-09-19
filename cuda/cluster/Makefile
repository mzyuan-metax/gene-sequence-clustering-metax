all:
	nvcc -O3 -arch=sm_80 -I/opt/hpcc/ompi/include/ -L/opt/hpcc/ompi/lib -lmpi main.cpp func.cu -o cluster
