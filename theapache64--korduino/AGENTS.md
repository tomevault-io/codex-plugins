Sample test format
```kotlin
@ParameterizedTest
@ValueSource(strings = ["&&", "||"])
fun basic(operator: String) {
    val input = SourceFile.kotlin(
        "Main.kt",
        """
        fun main() : Int {
            val x = 1 > 3 $operator 2 < 4
            if (x) {
                println("Hello Kotlin!")
            }
            return 0
        }
    """.trimIndent(),
    )

    val actualOutput = generateAndCompileCppSourceCode(listOf(input)).readActualOutput(Arg.Platform.Target.STD_CPP)

    val expectedOutput = """
        #include <iostream>
        int main() {
            bool x = 1 > 3 $operator 2 < 4;
            if (x) {
                std::cout << "Hello Kotlin!" << std::endl;
            }
            return 0;
        }
        
    """.trimIndent()

    actualOutput.should.equal(expectedOutput)
}
```

---
> Source: [theapache64/korduino](https://github.com/theapache64/korduino) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-06 -->
