---
title: "Enforcing Code Standards Across Projects with NuGet Packages"
date: 2025-08-05
---

Maintaining consistent code quality across multiple projects and teams is one of the challenges in software development. Teams copy `.editorconfig` files between projects, each team tweaks their analyzer settings slightly differently, and before you know it, you have a dozen different "standards" across your organization. 

This post demonstrates how to solve this problem by packaging your code standards into a NuGet package, enabling centralized management and consistent enforcement across all your projects.

---

## Why Package Code Standards?

Packaging them as a NuGet package provides:

- **Centralized Management**: Update standards once, deploy everywhere
- **Version Control**: Track changes and roll back if needed
- **Easy Adoption**: Simple package reference for new projects
- **Consistency**: Ensure all teams use the same standards

---

## 1. Project Structure Overview
You can find the repository [here](https://github.com/ImTris/ImTris.BaseCodingStandards). The structure looks like:

```
ImTris.BaseCodingStandards/
├── ImTris.BaseCodingStandards/
│   ├── ImTris.BaseCodingStandards.csproj
│   ├── config/
│   │   ├── build
│   │   │   ├── ImTris.BaseCodingStandards.props
│   │   │   └── ImTris.BaseCodingStandards.targets
│   │   └── files
│   │       ├── BannedSymbols.txt
│   │       ├── Default.editorconfig
│   │       ├── Exclusions.editorconfig
│   │       └── SonarAnalyzerExclusions.editorconfig
│   └── ImTris.BaseCodingStandards.nuspec
│
├── LICENSE
├── ImTris.BaseCodingStandards.sln
└── README.md
```

---

## 2. Props and Targets files

MSBuild `.props` and `.targets` files are the core mechanism for distributing build configuration through NuGet packages:

- **`.props` files**: Define properties and are imported early in the build process, before project-specific settings
- **`.targets` files**: Define targets and items, imported late in the build process after project evaluation

When you reference a NuGet package, MSBuild automatically imports these files, allowing the package to modify build behavior without manual configuration.

### ImTris.BaseCodingStandards.props

This file configures C# language features, enables analyzers, and sets up quality gates:

```xml
<Project>
    <PropertyGroup>
        <!-- C# language settings -->
        <LangVersion>latest</LangVersion>
        <Nullable>enable</Nullable>
        <ImplicitUsings>enable</ImplicitUsings>

        <!-- Build settings -->
        <Deterministic>true</Deterministic>
        <Features>strict</Features>

        <!-- Code quality analysis -->
        <EnableNETAnalyzers>true</EnableNETAnalyzers>
        <AnalysisMode>Recommended</AnalysisMode>
        <AnalysisLevel>latest</AnalysisLevel>
        <EnforceCodeStyleInBuild>true</EnforceCodeStyleInBuild>
        <ReportAnalyzer>true</ReportAnalyzer>

        <!-- Suppress specific warnings -->
        <NoWarn>$(NoWarn)</NoWarn>

        <!-- Treat warnings as errors in CI builds -->
        <TreatWarningsAsErrors Condition="'$(ContinuousIntegrationBuild)' == 'true'">true</TreatWarningsAsErrors>

        <!-- Documentation -->
        <GenerateDocumentationFile>false</GenerateDocumentationFile>
    </PropertyGroup>
</Project>
```

- Enables latest C# language features and nullable reference types
- Configures strict build settings for deterministic builds
- Activates .NET analyzers with recommended analysis mode
- Treats warnings as errors in CI environments only

### ImTris.BaseCodingStandards.targets

This file registers configuration files and sets up banned symbols analysis:

```xml
<Project>
    <!-- Register the editorconfig files to the project -->
    <ItemGroup>
        <EditorConfigFiles Include="$(MSBuildThisFileDirectory)\..\files\*.editorconfig" />
    </ItemGroup>

    <!-- Banned Symbols -->
    <PropertyGroup>
        <IncludeDefaultBannedSymbols Condition="$(IncludeDefaultBannedSymbols) == ''">true</IncludeDefaultBannedSymbols>
    </PropertyGroup>

    <ItemGroup>
        <AdditionalFiles Include="$(MSBuildThisFileDirectory)\..\files\BannedSymbols.txt"
                         Condition="$(IncludeDefaultBannedSymbols) == 'true'"
                         Visible="false" />
    </ItemGroup>
</Project>
```

- Automatically includes all `.editorconfig` files from the package
- Registers banned symbols file for `Microsoft.CodeAnalysis.BannedApiAnalyzers`
- IncludeDefaultBannedSymbols allows opt-out

---

## 4. EditorConfig Standards

The package includes multiple `.editorconfig` files focused on analyzer rules and code quality enforcement:

- **Default.editorconfig**: Core C# formatting and style rules
- **Exclusions.editorconfig**: Project-wide rule suppressions for false positives
- **SonarAnalyzerExclusions.editorconfig**: SonarAnalyzer-specific exclusions for noisy rules

### Getting Started with Your Standards

1. **Generate a baseline**: Use Visual Studio or Rider to generate an `.editorconfig` from an existing project that closely matches your desired standards
2. **Team consensus**: Hold discussions with teams who will consume the package to ensure buy-in and address concerns
3. **Gradual rollout**: Start with formatting rules before introducing stricter analysis rules
4. **Document exceptions**: Clearly document why certain rules are suppressed to maintain transparency

---

## 5. Local testing

To test new rules and analyzers quickly you can locally test the nuget package.

```sh
# Build the package
nuget pack ImTris.BaseCodingStandards.nuspec -Version 1.0.0

# If you haven't yet create a local feed:
dotnet nuget add source ./ -n LocalFeed

# Depending on the platform the location varies. Check it with:
dotnet nuget list source
# Now you can drop the nupkg file generated inside that folder.
```

---

## Next Steps: Deploy it to your company NuGet feed and add Custom Analyzers

Once you've tested your standards package locally, deploy it to your organization's private NuGet feed for company-wide adoption.
Take it even further? Learn how to [write your own C# analyzers](https://learn.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/tutorials/how-to-write-csharp-analyzer-code-fix) to enforce organization-specific rules that go beyond what's available in existing analyzer packages.

Custom analyzers can enforce:
- Company-specific naming conventions
- Architecture constraints (e.g., dependency directions)
- Security patterns specific to your domain
- Performance best practices for your codebase

Add your custom analyzers as additional NuGet package references in your standards package to distribute them alongside your configuration.

---

## References

- [ImTris.BaseCodingStandards Repository](https://github.com/ImTris/ImTris.BaseCodingStandards)
- [Sharing Coding Style and Roslyn Analyzers](https://www.meziantou.net/sharing-coding-style-and-roslyn-analyzers-across-projects.htm)
- [SonarAnalyzer.CSharp Rules](https://github.com/SonarSource/sonar-dotnet/tree/master/analyzers/src/SonarAnalyzer.CSharp)
- [Enabling High Code Quality in .NET](https://milan.milanovic.org/post/enabling-high-code-quality-in-net/)
- [How to Write C# Analyzer and Code Fix](https://learn.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/tutorials/how-to-write-csharp-analyzer-code-fix)
