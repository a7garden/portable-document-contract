# OXI Document conformance corpus

`corpus.json` is the machine-readable index. Paths are relative to this directory.

Each implementation should copy a file or set case into an isolated temporary vault and assert the listed result. Operation cases apply the described deterministic action and compare bytes or diagnostics. A test is incomplete if it only parses the body; it must validate the envelope and expected diagnostic. Set cases are evaluated together in one vault.

Corpus revision 1 is a bootstrap, not an exhaustive test suite. It covers the core Reader classification and initial Writer/Mutator preservation laws; implementations must also test every normative requirement for their claimed role. Add a shared regression fixture whenever an implementation disagreement is found. A behavior-changing fixture update must accompany the normative specification change that justifies it.
