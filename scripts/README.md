# SBS Control Generation Scripts

This directory contains scripts for generating machine-readable representations of SBS controls and generated documentation views.

## generate_regulation_pages.py

Parses all markdown control files in the `benchmark/` directory and generates a `regulations/` docs section containing:
- a landing page with counts and links for each regulation
- one page per regulation listing the tagged controls grouped by benchmark section

### Usage

```bash
# From the project root
python3 scripts/generate_regulation_pages.py
```

This will generate Markdown files in the `regulations/` directory. These files are used by the VitePress site and are regenerated automatically during the docs build.

## generate_xml.py

Parses all markdown control files in the `benchmark/` directory and generates a single XML file containing all controls in a structured, machine-readable format.

### Usage

```bash
# From the project root
python3 scripts/generate_xml.py
```

This will generate `sbs-controls.xml` in the project root.

### Output Format

The script generates an XML file with the following structure:

```xml
<sbs_benchmark version="1.0.0" xmlns="https://securitybenchmark.dev/sbs/v1">
  <metadata>
    <title>Security Benchmark for Salesforce</title>
    <version>1.0.0</version>
    <total_controls>24</total_controls>
  </metadata>
  <controls>
    <category>
      <name>OAuth Security</name>
      <description>...</description>
      <control id="SBS-OAUTH-001">
        <title>...</title>
        <statement>...</statement>
        <description>...</description>
        <rationale>...</rationale>
        <audit_procedure>...</audit_procedure>
        <remediation>...</remediation>
        <default_value>...</default_value>
      </control>
      <!-- more controls -->
    </category>
    <!-- more categories -->
  </controls>
</sbs_benchmark>
```

### Version Management

The current SBS version is stored in the `VERSION` file at the project root. This file contains a single line with the version number (e.g., `0.1.0`).

The XML generation script automatically reads from this file, ensuring the version is always synchronized across:
- The VERSION file (source of truth)
- Generated XML metadata
- Git release tags

**To release a new version:**

1. Update the `VERSION` file with the new version number
2. Commit the change: `git commit -am "Release v0.1.0"`
3. Create and push a tag: `git tag v0.1.0 && git push origin v0.1.0`

### Automated Generation

The XML file is automatically generated when a version tag is pushed to GitHub. The GitHub Action reads the version from the tag name.

The GitHub Action (`.github/workflows/release.yml`) will:
1. Run the script to generate the XML
2. Create a GitHub Release
3. Attach the XML file to the release

### For Tooling Vendors

Security tooling vendors can consume the XML file directly from GitHub releases:

```
https://github.com/yourusername/sbs/releases/download/v1.0.0/sbs-controls.xml
```

Each control includes all necessary information:
- **Control ID**: Unique identifier (e.g., `SBS-OAUTH-001`)
- **Title**: Short control name
- **Statement**: One-sentence requirement
- **Description**: Detailed explanation
- **Rationale**: Why this control exists
- **Audit Procedure**: Step-by-step verification process
- **Remediation**: How to fix noncompliance
- **Default Value**: Salesforce's default behavior

### Development Workflow

1. Edit markdown files in `benchmark/`
2. Regenerate regulation pages locally: `python3 scripts/generate_regulation_pages.py`
3. Test XML generation locally: `python3 scripts/generate_xml.py`
4. Verify output looks correct
5. Commit markdown and documentation changes
6. When ready to release, create a version tag
7. GitHub Action automatically generates and publishes XML

