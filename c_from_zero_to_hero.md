# C Language from Zero to Hero

A book in markdown work in progress that will cover basics to advanced topics for programming in the C language.


# Developing a Generic Swap Function
## search.h
```c from search.h
/**
 * Interface for generic search routines and other misc generic functions such as helper functions.
 *
 */

//these are used to make for stronger typing at the interface
typedef void * vp_key;
typedef void * vp_array;

//generic swap of size bytes data
void gswap(vp_array array, vp_key key, size_t size);

// generic linear search on a structure of length length
//pass in a pointer to a compare function such as memcmp or a custom one.
int gbinary_search(const vp_array array, const vp_key key, size_t elem_size, const unsigned int length, int (*cmp)(const void *a, const void *b, size_t elem_size));

// generic binary search on a structure of length length
//pass in a pointer to a compare function such as memcmp or a custom one.
int glinear_search(const vp_array array, const vp_key key, size_t elem_size, const unsigned int length, int (*cmp)(const void *a, const void *b, size_t elem_size));
```
## search.c

```c from search.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <assert.h>

#include "search.h"

void gswap(vp_array array, vp_key key, size_t size)
{
	void *temp = malloc(size);

	assert(temp != NULL);

	memcpy(temp, array, size);
	memcpy(array, key, size);
	memcpy(key, temp, size);

	free(temp);
}

int gbinary_search(const vp_array array, const vp_key key, size_t elem_size, const unsigned int length, int (*cmp)(const void *a, const void *b, size_t elem_size))
{
	int low = 0, high = length - 1, index;
	
	while (low <= high) {
		index = (high + low) / 2;
		int result = cmp(key, array + index * elem_size, elem_size);

		if (result == 0)
			return index;
		else if (result < 0)
			high = index - 1;
		else
			low = index + 1;
	}
	return -1;
}

int glinear_search(const vp_array array, const vp_key key, size_t elem_size, const unsigned int length, int (*cmp)(const void *a, const void *b, size_t elem_size))
{
	for (int index = 0 ; index < length ; index++) {
		int result = cmp(key, array + index * elem_size, elem_size);
		if (result == 0)
			return index;
	}
	return -1;
}
```
## from testsearch.c
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <assert.h>

#include "search.h"

#define NOT_FOUND -1

void test_gswap(void)
{
	printf("entering test_gswap\n");
	{
		int a = -1, b = -2, atemp, btemp;
		atemp = a;
		btemp = b;

		printf("Before swap\n");
		printf("a=%d\tb=%d\n", a, b);

		gswap(&a, &b, sizeof(a));

		printf("After swap\n");
		printf("a=%d\tb=%d\n", a, b);

		assert(a == btemp);
		assert(b == atemp);
	}
	{
		unsigned int a = 1, b = 2, atemp, btemp;
		atemp = a;
		btemp = b;

		printf("Before swap\n");
		printf("a=%u\tb=%u\n", a, b);

		gswap(&a, &b, sizeof(a));

		printf("After swap\n");
		printf("a=%u\tb=%u\n", a, b);

		assert(a == btemp);
		assert(b == atemp);
	}
	{
		long a = 1, b = 2, atemp, btemp;
		atemp = a;
		btemp = b;
		
		printf("Before swap\n");
		printf("a=%ld\tb=%ld\n", a, b);
		
		gswap(&a, &b, sizeof(a));
		
		printf("After swap\n");
		printf("a=%ld\tb=%ld\n", a, b);
		
		assert(a == btemp);
		assert(b == atemp);
	}
	{
		float a = 1.1, b = 2.2, atemp, btemp;
		atemp = a;
		btemp = b;
		
		printf("Before swap\n");
		printf("a=%f\tb=%f\n", a, b);
		
		gswap(&a, &b, sizeof(a));
		
		printf("After swap\n");
		printf("a=%f\tb=%f\n", a, b);
		
		assert(a == btemp);
		assert(b == atemp);
	}
	{
		double a = 1.1, b = 2.2, atemp, btemp;
		atemp = a;
		btemp = b;
		
		printf("Before swap\n");
		printf("a=%lf\tb=%lf\n", a, b);
		
		gswap(&a, &b, sizeof(a));
		
		printf("After swap\n");
		printf("a=%lf\tb=%lf\n", a, b);
		
		assert(a == btemp);
		assert(b == atemp);
	}
	{
		struct testStruct{
			int item;
		} a = {1}, b = {2}, atemp, btemp;
		
		atemp = a;
		btemp = b;
		
		printf("Before swap\n");
		printf("a=%d\tb=%d\n", a.item, b.item);
		
		gswap(&a, &b, sizeof(a));
		
		printf("After swap\n");
		printf("a=%d\tb=%d\n", a.item, b.item);
		
		assert(a.item == btemp.item);
		assert(b.item == atemp.item);
	}
	printf("exiting test_gswap\n");
}

void test_glinear_search(void)
{
	printf("entering test_glinear_search\n");

	const int array[] = {0, 1, 2, 3, 4};
	const unsigned int length = sizeof(array) / sizeof(int);

	printf("Number of elements in array is: %d\n", length);

	for (int key = 0; key < length; key++) {
		assert(key == glinear_search((vp_array)array, (vp_key)&key, sizeof(key), length, memcmp));
	}
	//test out of bounds
	int key = length;
	assert(NOT_FOUND == glinear_search((vp_array)array, (vp_key)&key, sizeof(key), length, memcmp));

	printf("exiting test_glinear_search\n");
}

void test_gbinary_search(void)
{
	printf("entering test_gbinary_search\n");

	const int array[] = {0, 1, 2, 3, 4};
	const unsigned int length = sizeof(array) / sizeof(int);

	printf("Number of elements in array is: %d\n", length);	

	for(int key = 0; key < length; key++){
		assert(key == gbinary_search((vp_array)array, (vp_key)&key, sizeof(key), length, memcmp));
	}
	//test out of bounds
	int key = length;
	assert(NOT_FOUND == gbinary_search((vp_array)array, (vp_key)&key, sizeof(key), length, memcmp));

	printf("entering test_gbinary_search\n");
}

void test_search(void)
{
	printf("entering test_search\n");

	test_gswap();
	test_glinear_search();
	test_gbinary_search();

	printf("exiting test_search\n");
}
```
## main driver for test
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <assert.h>

#include "search.h"

void test_search(void);

int main(void) 
{
	test_search();
	return 0;
}
```
## Makefile for project
```c

default: main.c search.c
	gcc -Wall -Werror -g -o search_algs main.c search.c testsearch.c
```

# Creating a Generic Vector
```c from gvector.c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <assert.h>

#define INIT_SIZE 2

typedef struct vector{
	int vector_length;
	int vector_loc;
	int elem_size;
	int last_double_index;
	void *vector;
} Vector;

void new_vector(Vector *vp, int elem_size){

	assert(elem_size>0);
	vp->elem_size = elem_size;
	vp->vector_length = INIT_SIZE;
	vp->vector_loc = 0;
	vp->last_double_index = 0;
	
	vp->vector = (char *) malloc(INIT_SIZE * elem_size);
	assert(vp->vector != NULL);

	printf("Created new vector\n");

}

void add_elem(Vector *vp, void *elem_addr){

	if(vp->vector_loc == vp->vector_length){
		vp->vector_length *= 2;
		vp->last_double_index = vp->vector_loc;
		vp->vector = realloc(vp->vector, vp->vector_length * vp->elem_size);
		assert(vp->vector != NULL);
	}
	memcpy((char *) vp->vector+vp->vector_loc*vp->elem_size, elem_addr, vp->elem_size);
	printf("adding value at location %d within %d byte array\n", vp->vector_loc, vp->vector_length);
	vp->vector_loc++;
}

void remove_elem(Vector *vp){

	if((vp->vector_loc == vp->last_double_index) && (vp->vector_loc != 0 )){
		vp->vector = realloc(vp->vector, vp->last_double_index * vp->elem_size);
		assert(vp->vector != NULL);
		vp->vector_length = vp->last_double_index;
	}
	
	if(vp->vector_loc > 0){	
		printf("Removing element\n");
		vp->vector_loc--;
	}
	else
		printf("Nothing to remove...at beginning of vector.\n");
			
}
		
void print_vector(Vector *vp){

	printf("Vector data is: \n");

	for (int i = 0; i < vp->vector_loc; i++){
		//assume it's an int for now--hence casting to int *!
		printf("%d\n", *( (int *)((char *) vp->vector+i*vp->elem_size)));
	}
}

void del_vector(Vector *vp){

	free(vp->vector);

}

int main(){

	Vector v;
	new_vector(&v, sizeof(int));

	for (int i = 0; i < 20; i++){
		add_elem(&v, &i);
	}
	for (int i = 0; i < 10; i++)
		remove_elem(&v);
	
	for (int i = 0; i < 10; i++){
		add_elem(&v, &i);
	}

	print_vector(&v);
	del_vector(&v);

}
```
