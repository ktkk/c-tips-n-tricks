# C Tips and Tricks

## Macro fun

### Fail macros
```c
#include <stdio.h>
#define FAIL_IF(EXP) ({if(EXP) { exit(EXIT_FAILURE); }})
#define FAIL_IF_MSG(EXP, MSG) ({if(EXP) { printf(MSG "\n"); exit(EXIT_FAILURE); }})
```

### Min Max macros
```c
#define MIN(A, B) ({ \
		    typeof(A) _a = (A); \
		    typeof(B) _b = (B); \
		    _a < _b ? _a : _b; \
		   })

#define MAX(A, B) ({ \
		    typeof(A) _a = (A); \
		    typeof(B) _b = (B); \
		    _a > _b ? _a : _b; \
		   })
```

### Quoted string from macro argument
```c
#include <stdio.h>

#define exprprint(expr) printf(#expr " = %d\n", expr)

int main() {
	int x = 6, y = 2;
	exprprint(x/y); /* expands to: printf("x/y" " = %d\n ", x/y) */
}
```

### Concatenate macro arguments
```c
#include <stdio.h>

#define CAT(first, second) first ## second

int main() {
	int x = 6, y = 2;
	int CAT(x, y) = 8; /* expands to: int xy = 8 */
}
```

### Repetitive function declaration macro expansion
```c
#define DECL_FN(name)\
	void func_##name() {\
		/*...*/\
	}

/* examples */
DECL_FN(destroy)
DECL_FN(render)
DECL_FN(update)
```

## String fun

### Substrings
```c
#include <stdio.h>

const char* string = "helloWorld";
int main() {
	printf("String but only starting from the 6th character: %s", &string[5]); /* Prints "World" */
}
```

## Struct magic

### Passing arrays around without them decaying into pointers with the use of structs
```c
typedef struct {
	int data[128];
} Container;

Container f(Container c) {
	return c;
}
```

### Union and struct stuff
```c
#include <stdio.h>

typedef struct{ float x; float y; } Vec2;

struct Entity {
	union {
		Vec2 position;
		struct { float x; float y; };
	};

	int size;
};

int main() {
	struct Entity entity1;

	Vec2 v2 = { 0.5f, 0.5f };

	entity1.position = v2;
	printf("struct: x = %f, y = %f\n", entity1.position.x, entity1.position.y);

	entity1.x = 1.f;
	entity1.y = 2.f;
	printf("separate: x = %f, y = %f\n", entity1.x, entity1.y);
}
```

## Other useful C stuff

### Enums & designated initializers
```c
typedef enum {
	FIRST = 0,
	SECOND,
	THIRD,
	COUNT_POS,
} Positions;

const char* array[COUNT_POS] = {
	[FIRST] = "First",
	[SECOND] = "Second",
	[THIRD] = "Third",
};
```

### Expressions in function arguments
```c
#include <stdio.h>
int main() {
	unsigned int numItems = 9;
	int i;

	for(i = numItems; i >= 0; --i) {
		/* Use singular of "item" when count is 1 */
		printf("You have %d item%s\n", i, (i == 1) ? "" : "s");
	}
}
```

### Colored stdout/stderr
```c
#include <stdio.h>

#define RED "\e[0;31m"
#define YELLOWBG "\e[43m"
#define RESET "\e[0m"

int main() {
	printf(RED YELLOWBG "hello world\n" RESET);
	return 0;
}
```

### Type checking variable argument list in custom printf-style functions with gcc(/clang)
```c
#include <stdio.h>
#include <stdarg.h>

#if defined(__GNUC__) || defined(__clang__)
#define CHECK_PRINTF_FMT(a, b) __attribute__ ((format (printf, a, b)))
#else
#define CHECK_PRINTF_FMT(...)
#endif

CHECK_PRINTF_FMT(1, 2) int dummy_printf(const char* fmt, ...) {
	va_list args;
	va_start(args, fmt);

	int result = vprintf(fmt, args);
	va_end(args);
	return result;
}
```

## Useful snippets

### Progress bar
```c
#include <stdio.h>
#include <unistd.h>

#define PROG_BAR_LENGTH 30

void UpdateBar(int percentDone) {
	int numChars = percentDone * PROG_BAR_LENGTH / 100;

	printf("\r[");
	for(int i = 0; i < numChars; ++i) {
		printf("#");
	}
	for(int i = 0; i < PROG_BAR_LENGTH - numChars; ++i) {
		printf("-");
	}
	printf("] %d%% Done", percentDone);
	fflush(stdout);
}
int main() {
	for (int i = 0; i <= 100; ++i) {
		UpdateBar(i);
		usleep(20000);
	}
	printf("\n");
}
```
