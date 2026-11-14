**Header record (H)**

|Columns|Content|
|---|---|
|1|`H`|
|2-7|Program name (6 characters, padded with spaces)|
|8-13|Starting address of the program (hex)|
|14-19|Length of the program in bytes (hex)|

**Text record (T)**

| Columns | Content                                                  |
| ------- | -------------------------------------------------------- |
| 1       | `T`                                                      |
| 2-7     | Starting address of the object code in this record (hex) |
| 8-9     | Length of the object code in this record, in bytes (hex) |
| 10-69   | Object code, up to 60 hex digits (30 bytes)              |

**End record (E)**

|Columns|Content|
|---|---|
|1|`E`|
|2-7|Address of the first executable instruction (hex)|

**Modification record (M), SIC/XE only**

|Columns|Content|
|---|---|
|1|`M`|
|2-7|Starting address of the field to be modified (hex)|
|8-9|Length of the field in half-bytes (hex)|