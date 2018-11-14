LIBS=-lcppcms -lbooster
INCLUDE=-I/usr/local/include -L/usr/local/lib

all: hello

hello: hello.cpp
	$(CXX) -O2  -Wall -g hello.cpp ${INCLUDE} -o hello ${LIBS}

clean:
	rm -fr hello hello.exe cppcms_rundir