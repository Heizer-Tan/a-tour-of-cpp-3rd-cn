# 1.10 建议

以下是第一章中最重要的建议，按主题整理：
[1] Don’t panic! All will become clear in time; §1.1; [CG: In.0].
[2] Don’t use the built-in features exclusively. Many fundamental (built-in)
features are usually best used indirectly through libraries, such as the 
ISO C++ standard library (Chapters 9–18); [CG: P.13].
[3] #include or (preferably) import the libraries needed to simplify
programming; §1.2.1.
[4] You don’t have to know every detail of C++ to write good programs.
[5] Focus on programming techniques, not on language features.
[6] The ISO C++ standard is the final word on language definition issues;
§19.1.3; [CG: P.2].
[7] “Package” meaningful operations as carefully named functions; §1.3;
[CG: F.1].
[8] A function should perform a single logical operation; §1.3 [CG: F.2].
[9] Keep functions short; §1.3; [CG: F.3].
[10] Use overloading when functions perform conceptually the same task
on different types; §1.3.
[11] If a function may have to be evaluated at compile time, declare it
constexpr; §1.6; [CG: F.4].
[12] If a function must be evaluated at compile time, declare it consteval;
§1.6.
[13] If a function may not have side effects, declare it constexpr or consteval;
§1.6; [CG: F.4].
[14] Understand how language primitives map to hardware; §1.4, §1.7,
§1.9, §2.3, §5.2.2, §5.4.
[15] Use digit separators to make large literals readable; §1.4; [CG: NL.11].


[16] Avoid complicated expressions; [CG: ES.40].
[17] Avoid narrowing conversions; §1.4.2; [CG: ES.46].
[18] Minimize the scope of a variable; §1.5, §1.8.
[19] Keep scopes small; §1.5; [CG: ES.5].
[20] Avoid “magic constants”; use symbolic constants; §1.6; [CG: ES.45]. 
[21] Prefer immutable data; §1.6; [CG: P.10].
[22] Declare one name (only) per declaration; [CG: ES.10].
[23] Keep common and local names short; keep uncommon and nonlocal
names longer; [CG: ES.7].
[24] Avoid similar-looking names; [CG: ES.8].
[25] Avoid ALL_CAPS names; [CG: ES.9].
[26] Prefer the {}-initializer syntax for declarations with a named type; §1.4;
[CG: ES.23].
[27] Use auto to avoid repeating type names; §1.4.2; [CG: ES.11].
[28] Avoid uninitialized variables; §1.4; [CG: ES.20].
[29] Don’t declare a variable until you have a value to initialize it with;
§1.7, §1.8; [CG: ES.21].
[30] When declaring a variable in the condition of an if-statement, prefer the
version with the implicit test against 0 or nullptr; §1.8.
[31] Prefer range-for loops over for-loops with an explicit loop variable;
§1.7.
[32] Use unsigned for bit manipulation only; §1.4; [CG: ES.101] [CG:
ES.106].
[33] Keep use of pointers simple and straightforward; §1.7; [CG: ES.42].
[34] Use nullptr rather than 0 or NULL; §1.7; [CG: ES.47].
[35] Don’t say in comments what can be clearly stated in code; [CG: NL.1]. 
[36] State intent in comments; [CG: NL.2].
[37] Maintain a consistent indentation style; [CG: NL.4].