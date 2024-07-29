---
layout: page
title: C# - Migrate to SDK style project
parent: C#
---

# Migrate to SDK style project

If you want to migrate an old .net Framework project to the newer SDK style project, then you could do this manually step by step with creating an new empty project and move the file to this new SDK styled project. Or you just use some help from a tool: [Try-Convert](https://github.com/dotnet/try-convert)


## Installation

```batch
dotnet tool install -g try-convert
```

## Update

```batch
dotnet tool update -g try-convert
```

## .NET 5.0 SDK

You need the .NET 5.0 SDK to be able to run the tool at this point:
https://dotnet.microsoft.com/en-us/download/dotnet/thank-you/sdk-5.0.408-windows-x64-installer 


## Convert

```batch
try-convert
```

This converts the old csporj to the newer SDK style format.
You could now even further delete old entries and reduce unneeded lines to a bare minimum.

**Exampel:**

[![Example](/assets/images/articles/migrate-sdk-project/example.png)](/assets/images/articles/migrate-sdk-project/example.png)

**Old project file:**

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="15.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <Import Project="$(MSBuildExtensionsPath)\$(MSBuildToolsVersion)\Microsoft.Common.props" Condition="Exists('$(MSBuildExtensionsPath)\$(MSBuildToolsVersion)\Microsoft.Common.props')" />
  <PropertyGroup>
    <Configuration Condition=" '$(Configuration)' == '' ">Debug</Configuration>
    <Platform Condition=" '$(Platform)' == '' ">AnyCPU</Platform>
    <ProjectGuid>{6B2BFAAB-717D-46E8-8FD7-E71EE01AFFCA}</ProjectGuid>
    <OutputType>Exe</OutputType>
    <RootNamespace>WindowsServiceEDU</RootNamespace>
    <AssemblyName>WindowsServiceEDU</AssemblyName>
    <TargetFrameworkVersion>v4.8</TargetFrameworkVersion>
    <FileAlignment>512</FileAlignment>
    <AutoGenerateBindingRedirects>true</AutoGenerateBindingRedirects>
    <Deterministic>true</Deterministic>
    <PublishUrl>publish\</PublishUrl>
    <Install>true</Install>
    <InstallFrom>Disk</InstallFrom>
    <UpdateEnabled>false</UpdateEnabled>
    <UpdateMode>Foreground</UpdateMode>
    <UpdateInterval>7</UpdateInterval>
    <UpdateIntervalUnits>Days</UpdateIntervalUnits>
    <UpdatePeriodically>false</UpdatePeriodically>
    <UpdateRequired>false</UpdateRequired>
    <MapFileExtensions>true</MapFileExtensions>
    <ApplicationRevision>0</ApplicationRevision>
    <ApplicationVersion>1.0.0.%2a</ApplicationVersion>
    <IsWebBootstrapper>false</IsWebBootstrapper>
    <UseApplicationTrust>false</UseApplicationTrust>
    <BootstrapperEnabled>true</BootstrapperEnabled>
  </PropertyGroup>
  <PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Debug|AnyCPU' ">
    <PlatformTarget>AnyCPU</PlatformTarget>
    <DebugSymbols>true</DebugSymbols>
    <DebugType>full</DebugType>
    <Optimize>false</Optimize>
    <OutputPath>bin\Debug\</OutputPath>
    <DefineConstants>DEBUG;TRACE</DefineConstants>
    <ErrorReport>prompt</ErrorReport>
    <WarningLevel>4</WarningLevel>
  </PropertyGroup>
  <PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Release|AnyCPU' ">
    <PlatformTarget>AnyCPU</PlatformTarget>
    <DebugType>pdbonly</DebugType>
    <Optimize>true</Optimize>
    <OutputPath>bin\Release\</OutputPath>
    <DefineConstants>TRACE</DefineConstants>
    <ErrorReport>prompt</ErrorReport>
    <WarningLevel>4</WarningLevel>
  </PropertyGroup>
  <ItemGroup>
    <Reference Include="System" />
    <Reference Include="System.Configuration.Install" />
    <Reference Include="System.Core" />
    <Reference Include="System.Management" />
    <Reference Include="System.ServiceProcess" />
    <Reference Include="System.Xml.Linq" />
    <Reference Include="System.Data.DataSetExtensions" />
    <Reference Include="Microsoft.CSharp" />
    <Reference Include="System.Data" />
    <Reference Include="System.Net.Http" />
    <Reference Include="System.Xml" />
  </ItemGroup>
  <ItemGroup>
    <Compile Include="Program.cs" />
    <Compile Include="ProjectInstaller.cs">
      <SubType>Component</SubType>
    </Compile>
    <Compile Include="ProjectInstaller.Designer.cs">
      <DependentUpon>ProjectInstaller.cs</DependentUpon>
    </Compile>
    <Compile Include="WindowsServiceEDU.cs">
      <SubType>Component</SubType>
    </Compile>
    <Compile Include="WindowsServiceEDU.Designer.cs">
      <DependentUpon>WindowsServiceEDU.cs</DependentUpon>
    </Compile>
  </ItemGroup>
  <ItemGroup>
    <None Include="App.config" />
  </ItemGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.Extensions.DependencyInjection">
      <Version>7.0.0</Version>
    </PackageReference>
  </ItemGroup>
  <ItemGroup>
    <EmbeddedResource Include="ProjectInstaller.resx">
      <DependentUpon>ProjectInstaller.cs</DependentUpon>
    </EmbeddedResource>
  </ItemGroup>
  <ItemGroup>
    <BootstrapperPackage Include=".NETFramework,Version=v4.8">
      <Visible>False</Visible>
      <ProductName>Microsoft .NET Framework 4.8 %28x86 and x64%29</ProductName>
      <Install>true</Install>
    </BootstrapperPackage>
    <BootstrapperPackage Include="Microsoft.Net.Framework.3.5.SP1">
      <Visible>False</Visible>
      <ProductName>.NET Framework 3.5 SP1</ProductName>
      <Install>false</Install>
    </BootstrapperPackage>
  </ItemGroup>
  <ItemGroup>
    <Folder Include="Properties\" />
  </ItemGroup>
  <Import Project="$(MSBuildToolsPath)\Microsoft.CSharp.targets" />
</Project>
```

**new SDK project file:**

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net48</TargetFramework>
    <OutputType>Exe</OutputType>
  </PropertyGroup>

  <ItemGroup>
    <Reference Include="System.Configuration.Install" />
    <Reference Include="System.ServiceProcess" />
  </ItemGroup>

  <ItemGroup>
    <Compile Update="ProjectInstaller.cs">
      <SubType>Component</SubType>
    </Compile>
    <Compile Update="ProjectInstaller.Designer.cs">
      <DependentUpon>ProjectInstaller.cs</DependentUpon>
    </Compile>
    <Compile Update="WindowsServiceEDU.cs">
      <SubType>Component</SubType>
    </Compile>
    <Compile Update="WindowsServiceEDU.Designer.cs">
      <DependentUpon>WindowsServiceEDU.cs</DependentUpon>
    </Compile>
  </ItemGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.Extensions.DependencyInjection" Version="7.0.0" />
    <PackageReference Include="Microsoft.Extensions.Hosting" Version="7.0.1" />
    <PackageReference Include="Microsoft.Extensions.Logging" Version="7.0.0" />
    <PackageReference Include="Serilog.Extensions.Hosting" Version="7.0.0" />
    <PackageReference Include="Serilog.Extensions.Logging" Version="7.0.0" />
    <PackageReference Include="Serilog.Sinks.Console" Version="4.1.0" />
    <PackageReference Include="Serilog.Sinks.Debug" Version="2.0.0" />
  </ItemGroup>

  <ItemGroup>
	  <Reference Include="System.Configuration.Install" />
  </ItemGroup>

  <ItemGroup>
    <EmbeddedResource Update="ProjectInstaller.resx">
      <DependentUpon>ProjectInstaller.cs</DependentUpon>
    </EmbeddedResource>
  </ItemGroup>
  
</Project>
```