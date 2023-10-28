# Itt.SimpleMsixProj

[![Nuget](https://img.shields.io/nuget/v/Itt.SimpleMsixProj)](https://www.nuget.org/packages/Itt.SimpleMsixProj)

Provides targets for a project which generates an msix using makeappx and dotnet publish.

## Usage

Download [Simple MSIX Setup Project.zip](https://github.com/mgaffigan/Itt.SimpleMsixProj/blob/master/Simple%20MSIX%20Setup%20Project.zip) 
to `Documents\Visual Studio 2022\Templates\ProjectTemplates`.  Once the template is installed:

1. Add a new project using the `Simple MSIX Setup Project` template.
1. Edit the project file `PublishProject` item to reference the appropriate source project(s).
1. Update the name, version, and display name of the application in `AppxManifest.xml`
1. Update the icons in the Assets folder
1. Import the `codesign.cer` certificate to the `Machine > Trusted People` store or update it to an existing trusted certificate
1. Build the project
1. Install the msix from the output folder

## Manual usage

Add a setup project with a .msbuildproj or .proj:

    <Project Sdk="Itt.SimpleMsixProj/2.0.0">
        <PropertyGroup>
            <TargetFramework>net7</TargetFramework>
            <SigntoolPfx>codesign_123.pfx</SigntoolPfx>
            <SigntoolPassword>123</SigntoolPassword>
        </PropertyGroup>

        <ItemGroup>
            <PublishProject Include="..\Itp.AmazonConnect.DesktopClient\Itp.AmazonConnect.DesktopClient.csproj" />
            <PublishProject Include="..\Itp.AmazonConnect.HidCallControl\Itp.AmazonConnect.HidCallControl.csproj">
                <PackagePath>Modules\Itp.AmazonConnect.HidCallControl\</PackagePath>
            </PublishProject>
            <PublishProject Include="..\Itp.AmazonConnect.InfluxDbPush\Itp.AmazonConnect.InfluxDbPush.csproj">
                <PackagePath>Modules\Itp.AmazonConnect.InfluxDbPush\</PackagePath>
            </PublishProject>
            <PublishProject Include="..\Itp.AmazonConnect.LdapContactsUI\Itp.AmazonConnect.LdapContactsUI.csproj">
                <PackagePath>Modules\Itp.AmazonConnect.LdapContactsUI\</PackagePath>
            </PublishProject>
            <PublishProject Include="..\Itp.AmazonConnect.RingAll\Itp.AmazonConnect.RingAll.csproj">
                <PackagePath>Modules\Itp.AmazonConnect.RingAll\</PackagePath>
            </PublishProject>
        </ItemGroup>
    </Project>

Add an AppxManifest.xml and Assets folder. See [SampleApp.Setup](https://github.com/mgaffigan/Itt.SimpleMsixProj/tree/master/SampleApp.Setup) for example project. Build and find the msix in the output folder.

## Items

Loose files may be added in the `Assets` folder.  These will be copied to the `Assets` folder in the msix.
Files can be added to arbitrary paths with `OtherAsset` item.

    <ItemGroup>
        <OtherAsset Include="license.txt" />
    </ItemGroup>

Projects may be published into the package using `PublishProject`.  By default, the referenced project will
deploy to the root of the package.  If `PackagePath` is specified, the project will be in the specified path
within the project.

	<ItemGroup>
		<PublishProject Include="..\Itp.AmazonConnect.InfluxDbPush\Itp.AmazonConnect.InfluxDbPush.csproj">
			<PackagePath>Modules\Itp.AmazonConnect.InfluxDbPush\</PackagePath>
		</PublishProject>
	</ItemGroup>

By default, the the publish will occur using framework-dependent deployment for win-x86.  This can be changed
using the project properties `RuntimeIdentifier` and `PublishCommandOptions`.

    <PropertyGroup>
        <TargetFramework>net7</TargetFramework>
        <RuntimeIdentifier>win-arm</RuntimeIdentifier>
        <PublishCommandOptions>--self-contained true</PublishCommandOptions>
    </PropertyGroup>

Individual projects can have additional properties passed to `dotnet publish` using `PublishOptions` metadata 
on `PublishProject`: 

	<ItemGroup>
		<PublishProject Include="..\Itp.AmazonConnect.InfluxDbPush\Itp.AmazonConnect.InfluxDbPush.csproj">
			<PackagePath>Modules\Itp.AmazonConnect.InfluxDbPush\</PackagePath>
			<PublishOptions>-p Version=1.2.3</PublishOptions>
		</PublishProject>
	</ItemGroup>

## Signing

If a PFX and password are specified as `<SigntoolPfx>` and `<SigntoolPassword>` then the msix will be 
signed.  The `/Package/Identity/@Publisher` must match the subject of the PFX for signing to succeed.

Signing is not performed when building in `Configuration=Release`.  It is recommended to use a tool
like the Windows Store or [msixhero](https://msixhero.net/) for release singing.

A sample self-signed certificate is included in the sample app (`codesign_123.pfx`).  The password is `123`.
This certificate may be used for testing, but you should use a properly issued certificate for production.
On the machine used for testing, import `codesign.cer` to the `Machine > Trusted People` 
store (`cert:\LocalMachine\TrustedPeople` in powershell).