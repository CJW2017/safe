
CC=g++
FLAGS=-g -Os -W -Wall -ansi
LDFLAGS=-L../.. -lbotan-1.9
CFLAGS=-I../../build/include

x509test: x509test.cpp
	$(CC) $(FLAGS) $(CFLAGS) x509test.cpp $(LDFLAGS) -o x509test

clean:
	rm -f x509test
