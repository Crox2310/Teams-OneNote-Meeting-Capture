# indexOf() Array Support Test Result

**Tested:** 26 September 2026
**Environment:** PA - Scratch Diagnostics
**Result:** NOT SUPPORTED

## Error received

"The template function 'indexOf' expects its first parameter to be of type string. The provided value is of type 'Array'."

## Conclusion

`indexOf(array, item())` does not work in this Power Automate environment. The varCandidateIndex counter variable (MM03b + MM04d) is the correct fallback and must be used in Scope_MultiMatch.

The build instructions have been updated to reflect this — MM04c uses `variables('varCandidateIndex')` not `indexOf()`.
