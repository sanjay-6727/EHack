# FOCA — Metadata and Document Footprinting

## 1. Purpose
FOCA (Fingerprinting Organizations with Collected Archives) is a Windows tool historically used to analyze public documents and extract metadata, such as author fields, software details, and document properties. Features and compatibility depend on the version.

**Memory hook:** FOCA = **look at the label on the file, not just its contents**.

## 2. Inputs
- A project/domain and document sources, or downloaded documents.
- File formats that contain metadata, such as PDF and Office documents.
- Internet access may be needed for online collection.

## 3. GUI workflow
1. Install from a trusted, official/project source and check compatibility with your Windows version.
2. Create a project and enter a domain or import documents, depending on the build.
3. Search/collect documents only within an authorized scope.
4. Analyze metadata and inspect fields such as author, creator application, timestamps, and paths.
5. Save findings and verify them against the original file.

## 4. Output and interpretation
Possible metadata includes author/creator, application version, creation/modification timestamps, document properties, and sometimes internal path/user-name clues. Exact fields vary by format and sanitization.

Explain: “FOCA extracted metadata embedded in the document. These fields can reveal information about document creation practices, but they may be absent, stale, editable, or inaccurate.”

## 5. Safe handling
Metadata may contain personal or confidential information. Do not publish extracted personal data. Remove unnecessary metadata from documents before public release using an approved workflow.

## 6. Troubleshooting
- No files: search indexing/source restrictions or unsupported document formats.
- No metadata: the document may have been sanitized or not store those fields.
- Tool fails to run: older .NET/Windows dependencies may be involved; use a trusted source and compatible environment.

## 7. Viva
**Q: What is metadata?** Data describing a file, such as author, creation time, or software.
**Q: Why is it useful in reconnaissance?** It may expose software, naming conventions, or internal workflow clues.
**Q: Is metadata always reliable?** No; fields can be removed or edited.
