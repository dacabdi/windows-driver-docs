---
title: INF Validation Errors and Warnings
description: Driver installation errors and warnings can appear as a result of the automatic INF verification that Microsoft Visual Studio performs.
ms.assetid: E021D8F8-BFDA-4F71-B8EA-0997096761FB
ms.date: 04/20/2017
ms.localizationpriority: medium
---

# INF Validation Errors and Warnings

This topic describes driver installation errors and warnings that can appear as a result of the automatic INF verification that Microsoft Visual Studio performs, or when you run the [InfVerif](infverif.md) tool.

Starting in Visual Studio 2015 with WDK 10, when you build your driver, the following INF file errors can appear in the Error List pane. If you are running InfVerif.exe from the command line, the tool displays these errors at the command prompt, or in the HTML version of the results.

## Error Guidance

InfVerif follows a general rule that the lower the error number, the more severe the issue.
Most error codes can be either a warning or an error depending on the arguments supplied to InfVerif.

### Handling Errors

You must fix all errors in order to pass driver tests on the Hardware Dev Center dashboard. Errors are related to the following conditions:

- The INF parser is unable to successfully interpret your INF
- The INF parser is able to interpret the INF only by making a default value assumption (ambiguous syntax)
- The arguments to InfVerif indicate that a rule set should be applied to the INF (such as Universal)

While you don't need to fix warnings before submitting your driver on the Dev Center, we recommend taking the time to understand the issue being reported. If you don't understand a given warning, your INF might not always behave as you expect.

Warnings are typically related to:

- Syntax that may be incorrect, but has valid scenarios where it is appropriate
- Syntax that is valid for the given InfVerif parameters but is an error in other modes, such as Universal

Issues related to the Universal setting appear as errors if:

- In Visual Studio, you build your driver with target platform set to **Universal** or **Mobile**.
- You run InfVerif.exe from the command line and specify the /u flag.

Issues related to the Universal setting appear as warnings if:

- In Visual Studio, you build your driver with target platform set to **Desktop**.
- You run InfVerif.exe from the command line and do not specify the /u flag.

## Error Codes

Error codes come in the following classifications:

- [Syntax in the INF file (1100-1299)](#syntax-in-the-inf-file-1100-1299)
- [Universal INF (1300-1319)](#universal-inf-1300-1319)
- [Installation (2000-2999)](#installation-2000-2999)

Not all error codes are listed below, as many have self-evident meanings. Errors in the 1000-1099 range are considered self-evident, as they are basic syntax errors.

## Syntax in the INF file (1100-1299)

While InfVerif failure means driver submission failure, driver installation may still succeed.
This is because when you install a driver, if errors are present in the INF file, Windows also tries the default value for the setting.
Windows does not fail driver installation due to errors in this range, but errors in this range indicate that the behavior may change depending on OS version or SKU.
In cases where the driver installs successfully, these errors indicate that there *are* circumstances where the driver may not install properly.

<table>
<thead>
<tr>
<th>Error Code</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>1100: DriverStore Copyfile name mismatch</strong></td>
<td>This error occurs when a file is copied or renamed from its original driver store name and location to a different name and location in the driver store.  For example:
<pre>
[SourceDisksFiles]
DriverFile.sys=1,x64  

[DestinationDirs]
CopyFileSection=13,SubDirectory  
  
[CopyFileSection]
DriverFile.sys
</pre>
The driver store maintains the original driver package directory structure.  In the code above, the original location of DriverFile.sys is <i>INF location</i>\x64, but the CopyFiles directive places it in <i>INF location</i>\SubDirectory.  The same error would be shown if the file was renamed as part of the copy.</td>
</tr>
<tr>
<td><strong>1203: Section not found</strong></td>
<td>For example, the following INF syntax causes error 1203:
<pre>
[MyInstallSection]
CopyFiles=driverFile.sys
</pre>
This error is reported because the <strong>CopyFiles</strong> directive expects a section name (that specifies the list of files to copy). However, the <strong>CopyFiles</strong> directive can specify a file name. To differentiate between a section name and a file name, preface a file name with the @ token as shown here:
<pre>
[MyInstallSection]
CopyFiles=@driverFile.sys
</pre>
</td>
</tr>
<tr>
<td><strong>1204: Provider cannot be Microsoft</strong></td>
<td>The Provider field in the [Version] section cannot specify Microsoft.
<pre>
[Version]
Signature="$Windows NT$"
Class=Sample
ClassGuid={78A1C341-4539-11d3-B88D-00C04FAD5171}
Provider="Microsoft"
</pre>
</td>
</tr>
<tr>
<td><strong>1205: Section [Driver_files] referenced from [Directive1] and [Directive2] directive</td>
<td>This warning is generated whenever two different directives point to the same section.
  <p>Note that while in most cases this is, indeed, an error, in some cases 1205 is reported even though the condition is on purpose.</td>
</tr>
<tr>
<td><strong>1212: Cannot have both [DefaultInstall] and [Manufacturer]</strong></td>
<td>A single INF cannot contain both [DefaultInstall] and [Manufacturer].  INFs authored with both should remove one of the two sections.
</td>
</tr>
<tr>
<td><strong>1220: Cannot directly reference a section defined in an included INF</strong></td>
<td>If your INF file references a <a href="https://docs.microsoft.com/windows-hardware/drivers/install/inf-ddinstall-section">DDInstall</a> section in an included INF, you must use the <strong>Needs</strong> directive. Any other directive that references a section from an included INF causes error 1220.
<p>In this example, the install section of A.INF references an equivalent install section in B.INF.</p>
<p>A.INF contains:</p>
<div class="code">
<pre>
A.INF
[InstallSectionA]
Include = B.INF
Needs = InstallSectionB
AddReg = AddRegB ; WARNING 1220

[InstallSectionA.Services]
Include = B.INF
Needs = InstallSectionB.Services
</pre>
<p>B.INF contains:</p>
<pre>
B.INF
[InstallSectionB]
AddReg = AddRegB
[InstallSectionB.Services]
...

[AddRegB]
...
</pre>
</div>
<p>The <strong>Needs</strong> directive must reference an equivalent install section to process in the current install section. For example, a Needs directive in [InstallSectionA.Services] should point to the .Services of another install section. The <strong>Needs</strong> directive may also be used to include the behavior of another DDInstall section of the same INF. Using the <strong>Needs</strong> directive on other types of sections may result in undesired behavior.</p></td>
</tr>
<tr>
<td><strong>1221: Cannot modify services regkey, must use HKR</strong></td>
<td>This error indicates that the INF file references a location in the services registry key, for example <strong>HKLM\SYSTEM\CurrentControlSet\Services&lt;em&gt;Service Name</em></strong>. When accessing the services key, you should instead use the relative root (<strong>HKR</strong>) to associate the registry value with the device or driver instance.
<p>When you use <strong>HKR</strong>, the registry value will not be present until the device is installed.</p></td>
</tr>
<tr>
<td><strong>1230: Missing file 'xxxx' under [SourceDisksFiles] section.</strong></td>
<td>This indicates that a file was specified as part of the driver package, but the source location of the file relative to the INF was not specified in a [SourceDisksFiles] section.
<pre>
[SourceDisksFiles]
filename=disk id
</pre>
Note that this error frequently occurs if architecture-decorated versions of [SourceDisksFiles] are specified (such as [SourceDisksFiles.amd64], but not all architectures supported by the INF have a [SourceDisksFiles] section.</td>
</tr>
<tr>
<td><strong>1233: Missing directive required for signature</strong></td>
<td>In the [Version] section, you must specify a CatalogFile directive (and associated catalog file) to receive a signature on a driver package.
<pre>
CatalogFile=wudf.cat
</pre>
</td>
</tr>
<tr>
<td><strong>1235: String token not defined in [Strings]</strong></td>
<td>A specified string token has no definition in the [Strings] section. For example, the INF file specifies <em>%REG_DWORD%</em> in an <em>add-registry section</em> specified by an <a href="https://docs.microsoft.com/windows-hardware/drivers/install/inf-addreg-directive"><strong>AddReg</strong></a> directive, but there is no corresponding REG_DWORD = 0x00010001 in the <a href="https://docs.microsoft.com/windows-hardware/drivers/install/inf-strings-section">[Strings]</a> section.
<p>This error frequently occurs if your INF file specifies a registry value that contains an environment variable. For example:</p>
<pre>
[MyAddReg]
HKR,,DllPath,%SystemRoot%\System32\myDll.sys
</pre>
This line causes the INF parser to attempt to locate the token "SystemRoot" from the [Strings] section, rather than the intended behavior of storing the literal "%SystemRoot%" in the registry.  To use the literal value %SystemRoot% rather than perform a string replacement, use the escape sequence %%.
<pre>
[MyAddReg]
HKR,,DllPath,%%SystemRoot%%\System32\myDll.sys
</pre>
</td>
</tr>
<tr>
<td><strong>1285: Cannot specify [ClassInstall32] section for Microsoft-defined class.</strong></td>
<td>As of Windows 10, IHV-supplied INFs are not allowed to use a [ClassInstall32] in an INF of any Microsoft-defined class.
</td>
</tr>
<tr>
<td><strong>1296: Specified service not associated with hardware</strong></td>
<td>Starting in Windows 10, version 1809, this has changed from a Warning to an Error.  The .Services sections are required for each defined target OS.  This is good practice and applies to all INFs and not just 1809.  

If you were previously not including this section because you had no services, and were relying on Inbox driver services, then you may need to create a .Services section that references the Inbox INF’s service using a NEEDS and INCLUDES statement.  

For example:  An INF file would have the following .Services section for each OS target to resolve this error.

<pre>
[XXXXXXXX.Install.NTx86.Services]
Include=filename.inf
Needs=inf-section-name.Services
</pre>

For devices that do not require a function driver, the NULL driver can be specified as follows:
<pre>
AddService = ,2.
</pre>
<b>Only use this in the case where the INF is installing a non-functional device to specify it does not need a driver.</b>
</td>
</tr>
</tbody>
</table>

## Universal INF (1300-1319)

>[!IMPORTANT]
>Your driver INF file is universal if you do not get any errors or warnings with error number in the range 13*xx*.

The following errors and warnings are related to INF configurability:

<table>
<thead>
<tr>
<th>Error/Warning Code</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>1300: Found legacy</strong><em>Xxx</em></td>
<td>You will see this error if you use deprecated sections or directives such as <a href="https://docs.microsoft.com/windows-hardware/drivers/install/inf-logconfig-directive"><strong>LogConfig</strong></a> or <a href="https://docs.microsoft.com/windows-hardware/drivers/install/inf-ddinstall-coinstallers-section"><strong>DDInstall.CoInstallers</strong></a>.</td>
</tr>
<tr>
<td><strong>1301: Found legacy</strong><em>Xxx</em><strong>operation</strong><em>Xxx</em></td>
<td>You will see this error if you use deprecated sections or directives such as <a href="https://docs.microsoft.com/windows-hardware/drivers/install/inf-logconfig-directive"><strong>LogConfig</strong></a> or <a href="https://docs.microsoft.com/windows-hardware/drivers/install/inf-ddinstall-coinstallers-section"><strong>DDInstall.CoInstallers</strong></a>.</td>
</tr>
<tr>
<td><strong>1302: Found legacy</strong><em>Xxx</em><strong>operation for</strong><em>Xxx</em></td>
<td>This error occurs when the operation affects something external to the driver package, like deleting a service or deleting a file.</td>
</tr>
<tr>
<td><strong>1303: Found legacy operation defining co-installers</strong></td>
<td>Error 1303 indicates that an AddReg operation is specifying a coinstaller. For example:
<pre>
AddReg = HKR,,CoInstallers32,0x00010000,"MyCoinstaller.dll"
</pre>
</td>
</tr>
<tr>
<td><strong>1304: Found legacy operation using non-relative key</strong></td>
<td>Error 1304 indicates that a registry operation uses a registry root other than HKR.</td>
</tr>
<tr>
<td><strong>1305: Found legacy operation using appendable multi-sz value</strong></td>
<td>Error 1305 indicates that the INF deletes a value from a <strong>REG_MULTI_SZ</strong> or appends a value to an existing <strong>REG_MULTI_SZ</strong>.</td>
</tr>
<tr>
<td><strong>1306: Found legacy operation with non-system target path</strong></td>
<td>Error 1306 indicates that a file copy specifies a target that is not under %SystemRoot%.</td>
</tr>
<tr>
<td><strong>1310-1312: Incorrect section extension for a Needs directive</strong></td>
<td>Needs directives effectively do a copy/paste of the needed section into the referencing section.  As a baseline validation, InfVerif compares the extension of the section.  This means that a [DDInstall.Services] can only use the Needs directive on other [DDInstall.Services] sections.</td>
</tr>
<tr>
<td><strong>1313-1314: Missing includes directive</strong></td>
<td>In each section that uses a Needs directive, there must be a corresponding Includes directive to reference the INF that contains the target section.  Previously the Needs directive would be valid if the Include directive was in another INF section.</td>
</tr>
<tr>
<td><strong>133x: Functional errors</strong></td>
<td>Multiple registry sections write to a single global key. For example, different sections could have a service set to different service configurations, a global registry key set to different data values, or a destination file pointing to different source files.</td>
</tr>
</tbody>
</table>

## Installation (2000-2999)

Issues in the 2000-2999 range appear as warnings. Possible values include the following.

<table>
<thead>
<tr>
<th>Error Code</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>2083: Section not referenced or used</strong></td>
<td>This warning indicates that the INF file provides a section that is not referenced. When the driver is installed, the contents of the section referenced in the warning are not evaluated.</td>
</tr>
<tr>
<td><strong>2222: Legacy directive will be ignored.</strong></td>
<td>This warning indicates that the INF specifies a deprecated directive. When the driver is installed, the directive referencing the section is not evaluated. For example, the <a href="https://docs.microsoft.com/windows-hardware/drivers/install/inf-logconfig-directive"><strong>INF LogConfig Directive</strong></a> directive is no longer supported, so the following section results in this warning.
<pre>
[InstallSection.LogConfigOverride]
LogConfig=LogConfigSection
...
</pre>
For information about which INF directives are deprecated, see <a href="https://docs.microsoft.com/windows-hardware/drivers/install/inf-directives">INF Directives</a>.</td>
</tr>
<tr>
<td><strong>2223: Section should have an architecture decoration</strong></td>
<td>This warning indicates that the INF file contains an <a href="https://docs.microsoft.com/windows-hardware/drivers/install/inf-manufacturer-section"><strong>INF Manufacturer Section</strong></a> that specifies a <a href="https://docs.microsoft.com/windows-hardware/drivers/install/inf-models-section"><strong>model section</strong></a> with no architecture decoration. For example, the following INF syntax would result in warning 2223:
<pre>
[Manufacturer]
%MfgName% = InstallSection

[InstallSection]
...
</pre>
When you install the driver, the preceding INF syntax defaults to x86.
<p>Instead, declare all supported architectures and provide a corresponding install section for each:</p>
<pre>
[Manufacturer]
%MfgName% = InstallSection, NTX86, NTAMD64

[InstallSection.NTAMD64]
...

[InstallSection.NTX86]
...
</pre>
If the INF file specifies a decorated section for x86 and an undecorated section, the undecorated section is ignored when you install your driver.</td>
</tr>
</tbody>
</table>
