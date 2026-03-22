# Seamoth Docs

Edit docs in Markdown, export polished DOCX using the branded Word template.

## MacOS

Install:

```bash
brew install pandoc
```

Convert MD to DOCX:

```bash
pandoc docs/requirements/src/VBC_GLIDER_v1_Spec.md --from markdown --to docx --reference-doc docs/requirements/template/VBC_GLIDER_v1_Spec.docx -o docs/requirements/dist/VBC_GLIDER_v1_Spec.docx
```

## Windows

Install:

- [Pandoc](https://pandoc.org/installing.html)

```powershell
winget install --id JohnMacFarlane.Pandoc -e
```

Convert MD to DOCX:

```powershell
pandoc .\docs\requirements\src\VBC_GLIDER_v1_Spec.md --from markdown --to docx --reference-doc .\docs\requirements\template\VBC_GLIDER_v1_Spec.docx -o .\docs\requirements\dist\VBC_GLIDER_v1_Spec.docx
```
