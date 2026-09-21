# Docentric AX Attachments Bulk Upload Examples

This project contains examples of adding the Docentric AX **Bulk File Upload** control (`DocBulkFileUploadControl`) in your own Microsoft Dynamics 365 Finance and Operations (D365FO) forms.

The showcase model adds the control to standard forms in five ways:

- An Action Pane button that opens the **Bulk upload** dialog on the purchase order,
- A tab page on the purchase order header, using the review flow,
- A tab page on the purchase order line, using the review flow with a narrowed file policy,
- A FactBox form part with a custom access behavior, and
- A tab page on the released products where each .jpg, .jpeg, .png, .gif or .bmp file becomes a product image through a per-file handler (`IDocBulkFileUploadHandler`).

[Docentric AX](https://ax.docentric.com) is a complete reporting and output management solution for Microsoft Dynamics 365 Finance and Operations. The solution provides companies and partners with a better way to create reports and automate the distribution and archiving of business documents.

Two editions of the solution are available:

- Docentric AX Free Edition: Solves most common shortcomings in D365FO output management.
- Docentric AX Full Edition: Report design and advanced output management features.

A complete feature list with a comparison to standard D365FO solutions and what’s part of the Free and Full Edition is described here: https://ax.docentric.com/feature-list.

## Download

Each release contains the source archives and the model file for import with ModelUtil.
[Releases](../../releases)


## Articles

You can learn more about the solution here:
- [Bulk Upload Attachments in D365FO with Docentric](https://ax.docentric.com/<to-be-published>) <!-- TODO: replace with the published URL -->
- [Use the Docentric Bulk File Upload Control in Your Own D365FO Forms](https://ax.docentric.com/<to-be-published>) <!-- TODO: replace with the published URL -->

## Prerequisites

[Docentric AX Free Edition](https://ax.docentric.com/download/) v3.6.0 or later is required. The Bulk File Upload control is included in the Free Edition as well as in the Full Edition.

## Features

- **Ready-made dialog**: The **Attach files** button on the purchase order header calls `DocBulkFileUploadHelper::openBulkUploadDialog`, one call that opens the built-in Docentric **Bulk upload** dialog for the current record and passes the record-state lock (`isPurchTableAttachEligible`, resolved through `checkUpdate`) as its `_canCreate` argument.
- **Embedded control**: Two tab pages on the purchase order and one on the released product contain the control. Each one names its owner data source in `RefDataSourceName` (`PurchTable`, `PurchLine`, `InventTable`) and picks its `UploadFlow` (`ReviewBeforeUpload` for the tabs, `CompactUploadOnDrop` for the FactBox and the product image tab).
- **Per-control limits**: The order line tab narrows the policy to 10 MB per file and the `.pdf`, `.docx`, `.xlsx`, `.jpg` and `.png` file types; the header tab adds no limits of its own, so only the Document management parameters apply.
- **Access behaviors**: The embedded tabs use `DocBulkFileUploadDataSourceAccessBehavior`, which follows the Allow edit setting of the referenced data source. The FactBox data source does not allow editing, so `DocPurchOrderBulkUploadFactBox_DCBU` installs a `DocBulkFileUploadFixedAccessBehavior` from the purchase order's current business status on every `active()` instead. Both are set through `setAccessBehavior`.
- **Per-file handler**: `DocProductImageBulkUploadHandler_DCBU` implements `IDocBulkFileUploadHandler.processAttachment` and registers each uploaded image as a product image (`EcoResProductImage::create`), the same way the standard **Add image** button does. A file whose type cannot become an image stays a plain attachment and a warning is written to the Infolog. The handler is supplied with `setUploadHandler`.
- **Completion delegate**: `DocProductBulkUploadShowcase_DCBU` subscribes to `notifyAllComplete` and re-reads the host `InventTable` data source when at least one file was attached, so the **Product image** FactBox shows the new image at once.
- **Security**: The `DocentricAXBulkUploadControlShowcaseUser_DCBU` privilege grants access to the FactBox menu item. Attaching files still requires the standard permissions to attach documents to the record.

## Control properties

The design-time properties of `DocBulkFileUploadControl` (build class `DocBulkFileUploadBuild`), as shown in the Properties window of the form designer in Visual Studio. You can add the control to a tab page or a group.

| Property | Build class method | Type / values | Default | What it does |
|---|---|---|---|---|
| **DocuType** | `docuTypeId` | `DocuTypeId` | empty | Document type a file gets when the Document management parameters hold no default for its kind. Left empty, images take the default image type and other files the default file type from the parameters; where that is empty too, the user picks a type before the upload. |
| **Ref data source** | `refDataSourceName` | `str` | empty | Form data source whose current record owns the attachments. Left empty, the first data source of the form is used. A named data source that cannot be resolved is never replaced by the first one; nothing can be attached until it resolves. |
| **Max file size (MB)** | `maxFileSizeMB` | `real` | `0` | Largest single file, in MB. When both this and the Document management parameters limit are set, the smaller one applies. `0` means the parameters limit alone; when that is not configured either, the size is not limited. |
| **Allowed extensions** | `allowedExtensions` | `str`, comma-separated list (`.pdf,.docx,image/*`) | empty | Narrows the accepted file types for this control. Left empty, the file types enabled in Document management parameters apply, the same ones the platform enforces when it stores the file. |
| **Allow folder pick** | `allowFolders` | `NoYes` | `No` | Adds a **Browse folder** button that stages every file in a picked folder. Dropping a folder on the drop zone works either way. |
| **Max concurrent uploads** | `maxConcurrentUploads` | `int` | `0` (means 3) | Number of files uploaded at the same time. `1` uploads strictly one after another. `0` applies the built-in value of 3. |
| **Max file count** | `maxFileCount` | `int` | `0` (means 50) | Largest number of files in one batch; a file over the limit is rejected while picked and reported to the infolog. `0` applies the built-in value of 50; a batch is never unlimited. |
| **Add infolog entry on batch** | `addInfologEntry` | `NoYes` | `No` | Writes the outcome of a finished batch to the infolog: a summary line, the uploaded files and the failed ones. `No` suppresses only that end-of-batch entry; a file rejected before the upload starts is reported either way. |
| **Upload flow** | `uploadFlow` | `DocBulkFileUploadFlow`: `ReviewBeforeUpload`, `CompactUploadOnDrop` | `ReviewBeforeUpload` | `ReviewBeforeUpload` stages the dropped files in a list where the user sets a document type per file and then starts the upload. `CompactUploadOnDrop` uploads the files as they are dropped, in a compact layout without an upload button; a file whose document type cannot be resolved is rejected instead of queried. |
| **Show instruction text** | `showInstructionText` | `NoYes` | `No` | Shows the introduction text and the note about the file limits above the drop zone. `CompactUploadOnDrop` never shows them, so the property has no effect there. |
| **Enable game corner** | `enableGameCorner` | `NoYes` | `No` | Offers the **Take a break** button that opens the mini-games overlay. The shipped `DocBulkFileUploadDialog` sets it to `No` when the `DisableGameCorner` system setting is on. |
| **Hide header when embedded** | `hideHeaderWhenEmbedded` | `NoYes` | `No` | Leaves out the **Upload files** title of the compact layout, for a host tab page or FactBox that already carries a caption. `ReviewBeforeUpload` has no title of its own, so the property has no effect there. |

An unset `NoYes` property persists as `No`, so a fresh control shows no instruction text, no game corner and no infolog entry until you set them; the shipped **Bulk upload** dialog sets all three to `Yes`. The control also exposes run-time `FormProperty` accessors that a host can read but that are not offered in the designer: `effectiveMaxFileSizeMB`, `allowMultipleSelection` (always true), `documentTypeOptions`, `defaultFileDocType`, `defaultImageDocType`, `clientLabels`, `gameEngineStrings`, `gamesEnabled`, `instructionText`, `limitsNoteTemplate`, `canAttach` and `attachReason`.

## Extending the control

Three extension points let a host shape what happens around an upload without changing the control. Each one is used by a showcase class in this repository.

- `setUploadHandler(IDocBulkFileUploadHandler)` installs one per-file handler, built by the host and run for each file after it is attached, which is how the control on the product image tab registers an uploaded image as a product image.
- `setAccessBehavior(IDocBulkUploadAccessBehavior)` installs the object that decides whether the current record accepts attachments; two behaviors are included in Docentric AX: `DocBulkFileUploadDataSourceAccessBehavior` (asks `docuAllowEdit_W` on the owner table, or the owner data source's `allowEdit()`) and `DocBulkFileUploadFixedAccessBehavior` (returns a decision the host computed).
- The delegates `notifyBeforePostProcess`, `notifyAfterAttach` and `notifyAllComplete` let any number of subscribers, also from other models, react to a file before and after it is attached (both can veto through `DocBulkFileUploadEventResult`) and to a finished batch. The showcase classes in this repository show each of them in use.

## Instructions

The X++ metadata in the repository is under the Metadata folder, which contains all the objects needed. We need to make the model visible to D365FO (at **:\AOSService\PackagesLocalDirectory**) by creating [Symbolic links](https://en.wikipedia.org/wiki/NTFS_links) to the model folder on the filesystem.

> Using the described approach you can use multiple Git repositories (e.g., solutions or customizations) on the same D365FO OneBox environment.

To do so, follow the next instructions (before you start, make sure you have Visual Studio closed):

1. **Add the model to D365FO.** Run the following command in PowerShell as an Administrator. It creates a symbolic link in **:\AOSService\PackagesLocalDirectory** for the model folder of this repository (a model that is already there is skipped):

`.\AOS_CreateMetadataSymbolicLinks.ps1`

2. **Remove the model from D365FO when you no longer need it.** Run the following command in PowerShell as an Administrator. It deletes the symbolic link from **:\AOSService\PackagesLocalDirectory** and leaves the repository itself untouched:

`.\AOS_RemoveMetadataSymbolicLinks.ps1`

> NOTE: Delete only symbolic links, not the source folders!

### Using scripts with Unified Development Environment (UDE)
Both scripts detect the D365FO package's local folder, but with UDE environments, you have a custom metadata folder, which is set through the `'Visual Studio > Dynamics 365 Extension > Configure Metadata'` in setting `'Folder for your own custom metadata'`. To create a symbolic link, use `.\AOS_CreateMetadataSymbolicLinks.ps1 -PackagesLocalFolder "C:\CustomAOS\PackagesLocalDirectory"`.

> NOTE: To remove use the same parameter with script `.\AOS_RemoveMetadataSymbolicLinks.ps1 -PackagesLocalFolder "C:\CustomAOS\PackagesLocalDirectory"`.

## Folder structure

The repository starts with the following folder structure:

- `Metadata`: *Folder containing the D365FO model (`DocentricBulkUploadShowcase`, model name `Docentric AX Bulk Upload Control Showcase`)*
- `Projects`: *Folder containing Visual Studio files (e.g., solutions, projects gluing the D365FO model artifacts).*
- `AOS_CreateMetadataSymbolicLinks.ps1`: *Script to create the symbolic links to the AosService\PackagesLocalDirectory folder.*
- `AOS_RemoveMetadataSymbolicLinks.ps1`: *Script to remove the symbolic links from the AosService\PackagesLocalDirectory folder.*
- `.gitattributes`: *Defines project-based rules for manipulating files and paths attributes when a Git action is performed (e.g., line-endings, file compare, etc.).*
- `.gitignore`: *Defines project-based rules for ignoring files and folders in the Git repository.*

## Added controls

| Example | Form | Objects |
|---|---|---|
| **Attach files** Action pane button, opens the **Bulk upload** dialog | **Purchase order** (`PurchTable`) | `PurchTable.Extension_DCBU` (`AttachFilesButton_DCBU`), `DocPurchOrderBulkUploadShowcase_DCBU::attachFilesButtonClicked` |
| **Attachments upload** tab page on the order header, review flow | **Purchase order** (`PurchTable`) | `PurchTable.Extension_DCBU` (`TabHeaderBulkUpload_DCBU`, `HeaderBulkUploadControl_DCBU`), `RefDataSourceName` = `PurchTable` |
| **Attachments upload** tab page on the order line, review flow with a narrowed file policy | **Purchase order** (`PurchTable`) | `PurchTable.Extension_DCBU` (`TabLineBulkUpload_DCBU`, `LineBulkUploadControl_DCBU`), `RefDataSourceName` = `PurchLine` |
| **Attachments upload** FactBox form part, compact drop flow with a fixed access behavior and a **View all attachments** button | **Purchase order** (`PurchTable`) | Form `DocPurchOrderBulkUploadFactBox_DCBU`, menu item `DocPurchOrderBulkUploadFactBox_DCBU`, control `FactBoxBulkUploadControl_DCBU`, privilege `DocentricAXBulkUploadControlShowcaseUser_DCBU` |
| **Product image attachments upload** tab page, each .jpg, .jpeg, .png, .gif or .bmp file becomes a product image | **Released product details** (`EcoResProductDetailsExtended`) | `EcoResProductDetailsExtended.Extension_DCBU` (`TabPageBulkUpload_DCBU`, `ProductBulkUploadControl_DCBU`), `DocProductBulkUploadShowcase_DCBU`, `DocProductImageBulkUploadHandler_DCBU`, `RefDataSourceName` = `InventTable` |

The tab pages are moved to the end of the header and line tab strips at runtime by `DocPurchOrderBulkUploadShowcase_DCBU::purchTableInitialized`. The labels of the model live in the `@DocentricBulkUploadShowcase` label file.

## License

This software is licensed under the **MIT License**. In short: you may use, copy, modify, merge, publish, distribute, sublicense and sell copies of it free of charge, as long as the copyright notice and the permission notice stay in all copies. The software is provided "as is", without warranty of any kind, and the authors are not liable for any claim or damage that arises from it. The full text is in the [LICENSE](LICENSE) file.
