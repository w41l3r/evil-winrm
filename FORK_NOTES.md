# Maintained fork notes

## Relationship to upstream

This repository is a maintained fork of [Hackplayers/evil-winrm](https://github.com/Hackplayers/evil-winrm). It preserves the upstream license, authorship, and Git history. The upstream project does not endorse or support the fork-specific behavior documented here.

The default `dev` branch tracks upstream development and carries a deliberately small behavioral difference. Upstream changes should be incorporated regularly while keeping that difference isolated and reviewable.

## Deliberate behavior difference

Upstream Evil-WinRM loads `Invoke-Binary`, `Dll-Loader`, and `Donut-Loader` from `menu` only after `Bypass-4MSI` succeeds. This fork loads those three built-in utilities on the first `menu` invocation, independently of the bypass state.

The divergence is intentional. It supports authorized environments where the operator does not want to run the bundled AMSI/ETW bypass or where that bypass is blocked, while still allowing the built-in utilities to be discovered and invoked.

## Security and detection trade-off

Removing the prerequisite does not bypass AMSI or any endpoint control. It causes the embedded PowerShell utility definitions to be sent before an AMSI bypass has necessarily been applied. Endpoint security products may inspect, log, or block those definitions.

Operators must decide whether to run `Bypass-4MSI` before `menu`, call `menu` directly, or avoid these utilities entirely. That decision belongs in the engagement plan and must remain within the supplied authorization.

## Included changes

- Load the built-in utilities once, on the first `menu` invocation, regardless of `@Bypass_4MSI_loaded`.
- Preserve the `@psLoaded` guard so the definitions are not repeatedly loaded.
- Retain upstream's LF/CRLF menu-output normalization fix.
- Retain upstream's improved error handling for blocked or failed AMSI/ETW bypass attempts.
- Document the fork-specific installation path and operational trade-off.

The original proposal and upstream decision are recorded in [Hackplayers/evil-winrm#84](https://github.com/Hackplayers/evil-winrm/pull/84).

## Validation baseline

The maintained difference was reviewed with the following offline checks:

- Ruby syntax and CLI version checks;
- LF and CRLF menu-normalization regression checks;
- PowerShell parsing of the embedded utility definitions;
- offline PowerShell loading and menu enumeration for `Dll-Loader`, `Donut-Loader`, and `Invoke-Binary`;
- RubyGem build validation.

These checks do not establish behavior against every Windows, PowerShell, AMSI, or endpoint-security version. No live-target result should be inferred from them.

## Installing this fork

The upstream RubyGem and Docker image do not contain the fork-specific behavior. Clone the maintained branch directly:

```sh
git clone --branch dev https://github.com/w41l3r/evil-winrm.git
cd evil-winrm
bundle install
bundle exec ruby evil-winrm.rb --help
```

Use Evil-WinRM only for explicitly authorized administration, security testing, or research.
