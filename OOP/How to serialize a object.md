```go
type FullObject struct {
    Flag       bool     // 1 byte  - Boolean
    Unsigned8  uint8    // 1 byte  - Unsigned 8-bit Integer
    Signed8    int8     // 1 byte  - Signed 8-bit Integer (Two's Complement)
    Unsigned16 uint16   // 2 bytes - Unsigned 16-bit Integer (Big-Endian)
    Signed16   int16    // 2 bytes - Signed 16-bit Integer (Big-Endian Two's Complement)
    Unsigned32 uint32   // 4 bytes - Unsigned 32-bit Integer (Big-Endian)
    Float32    float32  // 4 bytes - IEEE 754 Single-Precision Floating Point
    FixedBytes [4]byte  // 4 bytes - Raw binary byte array payload
    TextASCII  [4]byte  // 4 bytes - Fixed ASCII character byte array
}

func main() {
	obj := FullObject{
		Flag:       true,
		Unsigned8:  200,
		Signed8:    -42,
		Unsigned16: 1000,
		Signed16:   -1024,
		Unsigned32: 439041101,
		Float32:    -12.5,
		FixedBytes: [4]byte{0xDE, 0xAD, 0xBE, 0xEF},
		TextASCII:  [4]byte{'C', 'O', 'D', 'E'},
	}

	var buf bytes.Buffer
	_ = binary.Write(&buf, binary.BigEndian, obj)

	// Quick inspection: Raw hex stream
	fmt.Printf("Raw Hex: %X\n\n", buf.Bytes())

	// Formatted inspection: Formatted Hex Dump
	fmt.Print(hex.Dump(buf.Bytes()))
}
```

```output
Raw Hex: 01C8D603E8FC001A2B3C4DC1480000DEADBEEF434F4445

00000000  01 c8 d6 03 e8 fc 00 1a  2b 3c 4d c1 48 00 00 de  |........+<M.H...|
00000010  ad be ef 43 4f 44 45                              |...CODE|
```

## 2. Marshalling Field-by-Field (Object → Bytes)

| Field            | Input Value       | Encoding Mechanism & Paper Math                                                                                         | Binary Stream                         | Hex Bytes     | Size    |
| ---------------- | ----------------- | ----------------------------------------------------------------------------------------------------------------------- | ------------------------------------- | ------------- | ------- |
| **`Flag`**       | `true`            | Boolean rule: `0x00` = `false`, `0x01` = `true`                                                                         | `00000001`                            | `01`          | 1 byte  |
| **`Unsigned8`**  | `200`             | base10(200)=12×16+8=hex(C8)                                                                                             | `11001000`                            | `C8`          | 1 byte  |
| **`Signed8`**    | `-42`             | 8-bit Two's Complement (256−42=214=D6 in hex​)                                                                          | `11010110`                            | `D6`          | 1 byte  |
| **`Unsigned16`** | `1000`            | Big-Endian: ⌊1000/256⌋=3 (`0x03`), 1000 % 256=232 (`0xE8`)                                                              | `00000011 11101000`                   | `03 E8`       | 2 bytes |
| **`Signed16`**   | `-1024`           | 16-bit Two's Complement (65536−1024=64512=FC00 in hex​)                                                                 | `11111100 00000000`                   | `FC 00`       | 2 bytes |
| **`Unsigned32`** | `439041101`       | Direct Big-Endian split: `0x1A2B3C4D` across 4 bytes                                                                    | `00011010 00101011 00111100 01001101` | `1A 2B 3C 4D` | 4 bytes |
| **`Float32`**    | `-12.5`           | **IEEE 754:** Sign=1, Exp=3+127=130 (`10000010`), Mantissa=12.5 in base 10=1100.1 in base 2=1.1001​×2^3                 | `11000001 01001000 00000000 00000000` | `C1 48 00 00` | 4 bytes |
| **`FixedBytes`** | Raw Blob          | Direct literal byte copy                                                                                                | `11011110 10101101 10111110 11101111` | `DE AD BE EF` | 4 bytes |
| **`TextASCII`**  | `'C','O','D','E'` | ASCII Table Lookup:<br><br>  <br><br>`'C'` = 67 (`0x43`), `'O'` = 79 (`0x4F`), `'D'` = 68 (`0x44`), `'E'` = 69 (`0x45`) | `01000011 01001111 01000100 01000101` | `43 4F 44 45` | 4 bytes |

### Final Packed Output Stream (23 Bytes Total)

- **Hex Stream:** `01 C8 D6 03 E8 FC 00 1A 2B 3C 4D C1 48 00 00 DE AD BE EF 43 4F 44 45`
    
- **Binary Stream:** `00000001 11001000 11010110 00000011 11101000 11111100 00000000 00011010 00101011 00111100 01001101 11000001 01001000 00000000 00000000 11011110 10101101 10111110 11101111 01000011 01001111 01000100 01000101`


To restore the object, scan the 23-byte buffer sequentially using the schema's field byte counts