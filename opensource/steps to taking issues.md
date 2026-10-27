Before touching code, split issue into separate statement. Each one is either a fact to verify or a requirement to meet
Map the issue to the code: start from the output and trace backards
```
# search for the text the user sees
grep -rn "assertion(s) passed" --include=*.go .
grep -rn "matched in order" --include=*.go .
```

Learn the vocabularies
- the projects doc. `grep -rn -i "matcher" scenarios/README.md`