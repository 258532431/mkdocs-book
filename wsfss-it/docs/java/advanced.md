This section explains several conventions used in this documentation.

## Symbols

This documentation use some symbols for illustration purposes. Before you read on, please make sure you've made yourself familiar with the `following` list of conventions

=== "Java"

    ```java
    public class HelloWorld {
    }
    ```

=== "C++"

    ```c++
    #include <iostream>

    int main(void) {
    std::cout << "Hello world!" << std::endl;
    return 0;
    }
    ```

See additional configuration options:

- [SuperFences](https://www.baidu.com)

- [Tabbed](https://www.baidu.com)

## Usage

???+ tip

    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod
    nulla. Curabitur feugiat, tortor non consequat finibus, justo purus auctor
    massa, nec semper lorem quam in massa.

This configuration enables Markdown table support, which should normally be enabled by default, but to be sure, add the following lines to mkdocs.yml:

````yaml
markdown_extensions:
  - tables
````

## Tables

Data tables can be used at any ^^position in your project documentation^^ and can contain arbitrary Markdown, including inline code blocks, as well as icons and emojis:

| Method      | Description:material-information-outline:{ title="Important information" }                          |
| ----------- | ------------------------------------ |
| `GET`       | :material-check:     Fetch resource  |
| `PUT`       | :material-check-all: Update resource |
| `DELETE`    | :material-close:     Delete resource |

:smile: code:

``` py title="bubble_sort.py"
def bubble_sort(items):
    for i in range(len(items)):
        for j in range(len(items) - 1 - i):
            if items[j] > items[j + 1]:
                items[j], items[j + 1] = items[j + 1], items[j]
```

:arrow_up: [<font size="2">回到顶部</font>][top]

[top]: #