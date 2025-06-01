# C++: Length of an array

I have been away from coding vanilla C++ (without protos, vectors, containers) for so long that I had completely forgotten that one can't get the length of an array in C++ with `arr.length` like in Java. Took me back to my college days when I looked this up today:

```cpp
int length = sizeof(arr) / sizeof(arr[0]);
```

`sizeof(arr)` gives the size of the array in bytes. Java, [I think I'm in love with you!](https://tenor.com/en-GB/view/ted-mosby-love-im-in-gif-19020561)

Length of a character array is less verbose:

```cpp
char arr[] = "wow!";
int length = strlen(arr);
```
