# 2.6 建议

[1] Prefer well-defined user-defined types over built-in types when the
    built-in types are too low-level; §2.1.

[2] Organize related data into structures (structs or classes); §2.2; [CG:
    C.1].

[3] Represent the distinction between an interface and an implementation
    using a class; §2.3; [CG: C.3].

[4] A struct is simply a class with its members public by default; §2.3.

[5] Define constructors to guarantee and simplify initialization of classes;
    §2.3; [CG: C.2].

[6] Use enumerations to represent sets of named constants; §2.4; [CG:
    Enum.2].

[7] Prefer class enums over "plain" enums to minimize surprises; §2.4; [CG:
    Enum.3].

[8] Define operations on enumerations for safe and simple use; §2.4; [CG:
    Enum.4].

[9] Avoid "naked" unions; wrap them in a class together with a type field;
    §2.5; [CG: C.181].

[10] Prefer std::variant to "naked unions."; §2.5.
