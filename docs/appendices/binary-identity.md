# Analyzed Binary

Reproducible findings depend on identifying the exact input. The current local sample is not stored in this repository.

| Property | Value | Source |
|---|---|---|
| Project label | Dark Ages 4.21 Stone client | supplied identification |
| Filename | `Darkages.exe` | local sample |
| Size | 1,126,476 bytes | local sample measurement |
| SHA-256 | `36093dc1572521ea8c4c5f25548068c971a3d41c7722ef9fe0c05c94e24d6979` | local sample measurement |
| Windows file version | `360, 0, 0, 0` | Windows version resource |
| Windows product version | `360, 0, 0, 0` | Windows version resource |

## Open identity question

The working identification is 4.21 Stone client, while the executable's Windows version resource reports `360, 0, 0, 0`. The reason for this mismatch is unknown. Until another internal version marker, release artifact, or independently identified sample resolves it, references to 4.21 describe the project's supplied identification rather than a value proven from the executable itself.

## Reproducing the hash

On PowerShell:

```powershell
Get-FileHash -Algorithm SHA256 -LiteralPath 'C:\path\to\Darkages.exe'
```

Do not place the executable in this repository.
