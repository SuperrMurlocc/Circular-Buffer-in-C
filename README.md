# Circular-Buffer-in-C
A simple circular buffer implementation in C

### [circular_buffer.h](https://github.com/SuperrMurlocc/Circular-Buffer-in-C/blob/main/circular_buffer.h)
```c
#ifndef CIRCULAR_BUFFER_H
#define CIRCULAR_BUFFER_H

#include <stdio.h>
#include <stdlib.h>

struct circular_buffer_t {
    int *ptr;
    int begin;
    int end;
    int capacity;
    unsigned char full: 1;
};

int circular_buffer_create(struct circular_buffer_t *a, int N);
int circular_buffer_create_struct(struct circular_buffer_t **cb, int N);

void circular_buffer_destroy(struct circular_buffer_t *a);
void circular_buffer_destroy_struct(struct circular_buffer_t **a);

int circular_buffer_push_back(struct circular_buffer_t *cb, int value);
int circular_buffer_pop_front(struct circular_buffer_t *a, int *err_code);
int circular_buffer_pop_back(struct circular_buffer_t *a, int *err_code);

int circular_buffer_empty(const struct circular_buffer_t *a);
int circular_buffer_full(const struct circular_buffer_t *a);

void circular_buffer_display(const struct circular_buffer_t *a);

#endif CIRCULAR_BUFFER_H
```
<br>
<br>

### [circular_buffer.c](https://github.com/SuperrMurlocc/Circular-Buffer-in-C/blob/main/circular_buffer.c)

```c
#include "circular_buffer.h"

int circular_buffer_create(struct circular_buffer_t *cb, int N) {
    if (cb == NULL || N <= 0)
        return 1;

    cb->ptr = (int *) malloc(N * sizeof(int));

    if (cb->ptr == NULL)
        return 2;

    cb->capacity = N;
    cb->begin = 0;
    cb->end = 0;
    cb->full = 0;

    return 0;
}

int circular_buffer_create_struct(struct circular_buffer_t **cb, int N) {
    if (cb == NULL || N <= 0)
        return 1;

    *cb = (struct circular_buffer_t *) malloc(4 * sizeof(int) + sizeof(int *));

    if (*cb == NULL)
        return 2;

    int err = circular_buffer_create(*cb, N);
    if (err)
        free(*cb);

    return (err);
}

void circular_buffer_destroy(struct circular_buffer_t *cb) {
    if (cb == NULL)
        return;

    free(cb->ptr);
}

void circular_buffer_destroy_struct(struct circular_buffer_t **cb) {
    if (cb == NULL)
        return;

    circular_buffer_destroy(*cb);
    free(*cb);
}

int circular_buffer_push_back(struct circular_buffer_t *cb, int value) {
    if (cb == NULL || cb->capacity <= 0 || cb->begin >= cb->capacity || cb->end >= cb->capacity || cb->begin < 0 ||
        cb->end < 0)
        return 1;

    switch (cb->full) {
        case 0: {
            *(cb->ptr + (cb->end)) = value;

            cb->end = (cb->end + 1) % cb->capacity; 

            if (cb->end == cb->begin) { 
                cb->full = 1;
            }
            break;
        }
        case 1: {
            *(cb->ptr + (cb->end)) = value; 

            cb->end = (cb->end + 1) % cb->capacity; 
            cb->begin = (cb->begin + 1) % cb->capacity; 
            break;
        }
    }
    return 0;
}

int circular_buffer_pop_front(struct circular_buffer_t *cb, int *err_code) {
    if (cb == NULL || cb->capacity <= 0 || cb->begin >= cb->capacity || cb->end >= cb->capacity || cb->begin < 0 ||
        cb->end < 0) {
        if (err_code != NULL)
            *err_code = 1;
        return 0;
    }

    if (circular_buffer_empty(cb)) { 
        if (err_code != NULL)
            *err_code = 2;
        return 0;
    }

    int value;
    value = *(cb->ptr + cb->begin); 
    *(cb->ptr + cb->begin) = 0; 

    cb->begin = (cb->begin + 1) % cb->capacity; 
    if (circular_buffer_full(cb)) 
        cb->full = 0; 


    if (err_code != NULL)
        *err_code = 0;
    return value;
}

int circular_buffer_pop_back(struct circular_buffer_t *cb, int *err_code) {
    if (cb == NULL || cb->capacity <= 0 || cb->begin >= cb->capacity || cb->end >= cb->capacity || cb->begin < 0 ||
        cb->end < 0) {
        if (err_code != NULL)
            *err_code = 1;
        return 0;
    }

    if (circular_buffer_empty(cb)) {
        if (err_code != NULL)
            *err_code = 2;
        return 0;
    }

    int value;
    int last_element = (cb->end == 0) ? (cb->capacity - 1) : (cb->end - 1);
    value = *(cb->ptr + last_element);
    *(cb->ptr + last_element) = 0;

    cb->end = last_element;
    if (circular_buffer_full(cb))
        cb->full = 0;

    if (err_code != NULL)
        *err_code = 0;
    return value;
}

int circular_buffer_empty(const struct circular_buffer_t *cb) {
    if (cb == NULL || cb->capacity <= 0 || cb->begin >= cb->capacity || cb->end >= cb->capacity || cb->begin < 0 ||
        cb->end < 0)
        return -1;

    if (cb->end == cb->begin && !cb->full)
        return 1;
    else
        return 0;
}

int circular_buffer_full(const struct circular_buffer_t *cb) {
    if (cb == NULL || cb->capacity <= 0 || cb->begin >= cb->capacity || cb->end >= cb->capacity || cb->begin < 0 ||
        cb->end < 0)
        return -1;

    return cb->full;
}

void circular_buffer_display(const struct circular_buffer_t *cb) {
    if (cb == NULL || cb->capacity <= 0 || cb->begin >= cb->capacity || cb->end >= cb->capacity || cb->begin < 0 ||
        cb->end < 0)
        return;

    if (circular_buffer_empty(cb))
        return;

    int easy_case = cb->end > cb->begin;

    if (easy_case) {
        for (int i = cb->begin; i < cb->end; i++)
            printf("%d ", *(cb->ptr + i));
    } else {
        for (int i = cb->begin; i < cb->capacity; i++)
            printf("%d ", *(cb->ptr + i));
        for (int i = 0; i < cb->end; i++)
            printf("%d ", *(cb->ptr + i));
    }
}
```
