# Reading from STDIN
`stdin` is a basic stream used for reading input from the user in a c program. You can read input using the example below:
```c
char s[100];
scanf("%[^\n]%*c", s);
printf("%s", s);
```

# Data Types
A list of data types used in embedded c. See [[When to use uint32_t]] for more info on `size_t`
- `uint32_t`