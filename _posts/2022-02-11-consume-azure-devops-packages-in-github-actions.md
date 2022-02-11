---
layout: post
title: Consume packages from Azure DevOps Artifacts in GitHub Actions on Ubuntu
---

I need to consume some private packages published to Azure DevOps Artifacts, in a build on GitHub Actions running Ubuntu. While researching most answers either suggested to [install the credentials provider][1] or [use `nuget sources add` with a Personal Access Token (PAT)][2]. The first seemed like overkill for my situation so I opted for the latter. I already have this source configured in the `NuGet.config` file in the root of my solution, so instead of `add` I'll use `update`. It didn't work:

```yaml
- run: nuget sources update -Name XXX -Source https://pkgs.dev.azure.com/XXX -username "az" -password ${{ secrets.AZURE_DEVOPS_TOKEN }}
- run: nuget sources -Format detailed
- run: dotnet test
```

```
Run nuget sources update -Name XXX -Source https://pkgs.dev.azure.com/XXX -username "az" ***
  nuget sources update -Name XXX -Source https://pkgs.dev.azure.com/XXX -username "az" ***
  Package source "XXX" was successfully updated.

Run nuget sources -Format detailed
  nuget sources -Format detailed
  Registered Sources:
    1.  XXX [Enabled]
        https://pkgs.dev.azure.com/XXX
    2.  nuget.org [Enabled]
        https://api.nuget.org/v3/index.json

Run dotnet test
  dotnet test
  Determining projects to restore...
  /usr/share/dotnet/sdk/6.0.101/NuGet.targets(130,5): error : Unable to load the service index for source https://pkgs.dev.azure.com/XXX. [/home/runner/work/XXX.sln]
  /usr/share/dotnet/sdk/6.0.101/NuGet.targets(130,5): error :   Response status code does not indicate success: 401 (Unauthorized). [/home/runner/work/XXX.sln]
  Error: Process completed with exit code 1.
```

So even though the output of `nuget sources` lists my source, it cannot authenticate against it. I came across [this cross-platform issue in NuGet][3] causing the `nuget` command to write the config to `~/.config/NuGet` while `dotnet` was reading from `~/.nuget`. This as a result of `nuget` uses .NET Framework/Mono while `dotnet` is .NET Core. 🤷‍♂️

Well I actually don't need to update the "user" configuration, just for this build / this solution. So I'll just update the local `NuGet.config` instead using `-configfile`:

```yaml
- run: nuget sources update -Name XXX -Source https://pkgs.dev.azure.com/XXX -username "az" -password ${{ secrets.AZURE_DEVOPS_TOKEN }} -configfile NuGet.config
- run: dotnet test
```

```
Run dotnet test
  dotnet test
  Determining projects to restore...
  /usr/share/dotnet/sdk/6.0.101/NuGet.targets(130,5): error : Unable to load the service index for source https://pkgs.dev.azure.com/XXX. [/home/runner/work/XXX.sln]
  /usr/share/dotnet/sdk/6.0.101/NuGet.targets(130,5): error :   Password decryption is not supported on .NET Core for this platform. The following feed uses an encrypted password: 'XXX'. You can use a clear text password as a workaround. [/home/runner/work/XXX.sln]
  /usr/share/dotnet/sdk/6.0.101/NuGet.targets(130,5): error :   Encryption is not supported on non-Windows platforms. [/home/runner/work/XXX.sln]
  Error: Process completed with exit code 1.
```

Ah another surprise. `nuget` stores the password encrypted, but `dotnet` cannot decrypt it. I could use `-StorePasswordInClearText` at this point, but why not ditch the `nuget` command at all at this point?

```yaml
- run: dotnet nuget update source XXX --username "az" --password ${{ secrets.AZURE_DEVOPS_TOKEN }} --store-password-in-clear-text
- run: dotnet test
```
```
Run nuget sources update -Name XXX -Source https://pkgs.dev.azure.com/XXX -username "az" ***
  nuget sources update -Name XXX -Source https://pkgs.dev.azure.com/XXX -username "az" ***
  Package source "XXX" was successfully updated.
Run dotnet test
  dotnet test
  Determining projects to restore...
  Restored /home/runner/work/XXX.csproj (in 7.72 sec).
```

Success!

[1]: https://stackoverflow.com/a/61389452/58107
[2]: https://josh-ops.com/posts/authorize-azure-artifacts-in-github-actions/
[3]: https://github.com/NuGet/Home/issues/4413
