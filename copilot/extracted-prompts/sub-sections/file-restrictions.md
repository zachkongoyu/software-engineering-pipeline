# File Restrictions Section (UEe)

Injected dynamically when the user's working set contains files marked as
readonly. Rendered as a `<fileRestrictions>` XML block.

---

## Template

```
<fileRestrictions>
The following files are readonly. Making edits to any of these file paths
is FORBIDDEN. If you cannot accomplish the task without editing the files,
briefly explain why, but NEVER edit any of these files:
    - {{path/to/readonly/file1}}
    - {{path/to/readonly/file2}}
    ...
</fileRestrictions>
```

## Source Logic

Readonly URIs are collected from two sources:
1. **Working set entries** with `isMarkedReadonly === true`
2. **Chat variable attachments** with `isMarkedReadonly === true`

If zero readonly URIs exist, the section is not rendered at all (returns
empty fragment).
