<p align="center">
<img src="./Basemark.png" alt="Base logo" width="480" />
</p>

<!-- Badge row 1 - status -->

[![GitHub contributors](https://img.shields.io/github/contributors/base/docs)](https://github.com/base/docs/graphs/contributors)
[![GitHub commit activity](https://img.shields.io/github/commit-activity/w/base/docs)](https://github.com/base/docs/graphs/contributors)
[![GitHub Stars](https://img.shields.io/github/stars/base/docs.svg)](https://github.com/base/docs/stargazers)
![GitHub repo size](https://img.shields.io/github/repo-size/base/docs)
[![GitHub](https://img.shields.io/github/license/base/docs?color=blue)](https://github.com/base/docs/blob/main/LICENSE.md)

<!-- Badge row 2 - links and profiles -->

[![Website base.org](https://img.shields.io/website-up-down-green-red/https/base.org.svg)](https://base.org)
[![Blog](https://img.shields.io/badge/blog-up-green)](https://base.mirror.xyz/)
[![Docs](https://img.shields.io/badge/docs-up-green)](https://docs.base.org/)
[![Discord](https://img.shields.io/discord/1067165013397213286?label=discord)](https://base.org/discord)
[![Twitter Base](https://img.shields.io/twitter/follow/Base?style=social)](https://twitter.com/Base)

<!-- Badge row 3 - detailed status -->

[![GitHub pull requests by-label](https://img.shields.io/github/issues-pr-raw/base/docs)](https://github.com/base/docs/pulls)
[![GitHub Issues](https://img.shields.io/github/issues-raw/base/docs.svg)](https://github.com/base/docs/issues)

# Base Documentation Contributing Guide

Base Docs are community-managed. We welcome contributions from everyone to keep these docs accurate, helpful, and up to date.

> **Note:** This repository powers the public Base documentation site at [docs.base.org](https://docs.base.org). All documentation content lives under the `docs/` directory.

## Quick start

### Prerequisites

- Node.js v19 or higher

### Local development setup

1. **Clone the repository**
   ```bash
   git clone https://github.com/base/docs.git
   cd docs
   ```

2. **Install the Mint CLI**
   ```bash
   npm i -g mint
   ```

3. **Preview documentation locally**
   
   Navigate to the `docs/` directory (where `docs.json` is located) and run:
   ```bash
   cd docs
   mint dev
   ```
   
   Alternatively, run without a global install:
   ```bash
   npx mint dev
   ```

### Troubleshooting local preview

- Verify Node.js v19+ is installed: `node --version`
- Ensure you're running `mint dev` from the directory containing `docs.json` (typically `docs/`)
- If preview differs from production, update the CLI: `mint update`

## How to contribute

### Contribution workflow

1. **Fork and create a branch**: Fork the `base/docs` repository and create a descriptive branch for your changes (e.g., `fix-wallet-setup-guide` or `add-smart-contract-example`)

2. **Make your changes**: Edit content in the `docs/` directory following our structure and style guidelines below. Preview changes locally using the Mint CLI

3. **Submit a pull request**: Open a PR with a clear summary of your changes and links to related pages. Our docs team and community will review

> **Tip:** Submit small, focused PRs that address one topic or fix at a time. This speeds up review and makes it easier to iterate.

### What we're looking for

- **Corrections**: Fix typos, broken links, outdated information, or technical inaccuracies
- **Clarity improvements**: Simplify complex explanations, add missing context, or improve code examples
- **New guides**: Add tutorials or examples that fit within existing sections and serve clear Base use cases
- **Code updates**: Update examples to reflect current best practices or API changes

## Documentation structure

### Core principle: work within existing structure

> **Warning:** Do not create new top-level sections. All new content must fit within existing directories under `docs/`.

The Base documentation is organized into established sections:

- `get-started/` — Quick setup and onboarding
- `learn/` — Conceptual explanations and architecture
- `base-account/` — Base Account features and integration
- `base-app/` — Base App development guides
- `base-chain/` — Base chain technical details
- `cookbook/` — Use case-focused patterns and recipes
- `mini-apps/` — Mini app development guides
- `onchainkit/` — OnchainKit SDK documentation

Place your content in the most relevant existing section.

### Navigation and sidebar policy

> **Note:** We rarely modify the global navigation (top-level tabs) or create new sidebar sections. Focus on improving existing pages and adding content within current sections. Structural changes require strong justification and broad community benefit.

### Where to place your content

| Section | Purpose | Examples |
|---------|---------|----------|
| **Quickstart** | End-to-end setup to first success | "Deploy your first contract", "Connect your wallet" |
| **Concepts** | Explanations of components and architecture | "How Base Account works", "Gas optimization strategies" |
| **Guides** | Step-by-step tutorials for specific tasks | "Implementing social login", "Building a token swap" |
| **Examples** | Complete, runnable code demonstrating real usage | "NFT minting app", "DeFi dashboard" |
| **Reference** | API specs, method signatures, parameters | "OnchainKit API reference", "Smart contract interfaces" |
| **Cookbook** | Cross-cutting patterns and use cases | "Multi-chain deployment", "Testing strategies" |

#### Cookbook guidelines

- The `cookbook/` section is for use case-focused guides and reusable patterns, not product-specific documentation
- Prefer solutions that demonstrate building on Base across multiple tools and scenarios
- Focus on real-world problems developers commonly face

### Avoid subsection proliferation

> **Warning:** Keep the structure flat and scannable:
> - Place all guides at the same hierarchical level within their section
> - Organize Reference documentation by component or feature, not by use case
> - Use cross-links between pages instead of creating nested folder structures
> - If you need more than two levels of nesting, reconsider your approach

## Writing and formatting guidelines

### Writing style

1. **Be clear and concise**: Use active voice and address the reader directly ("you"). Remove unnecessary words
2. **Focus on the happy path**: Show the most common successful approach first. Mention alternatives only when relevant
3. **Use descriptive headings**: Make headings specific and scannable (prefer "Configure wallet connection" over "Setup")
4. **Maintain consistent terminology**: Use the same terms throughout. Define abbreviations on first use (e.g., "Smart Wallet Account (SWA)")
5. **Show, don't just tell**: Include working code examples wherever possible

### Writing for AI and human readers

Our documentation should be easily understood by both human developers and AI assistants that help them code.

**Best practices:**
- Use explicit, unambiguous language
- Link to related pages directly using descriptive anchor text
- Name libraries, tools, and versions explicitly (e.g., "OnchainKit v0.25" not "the latest version")
- Structure content with clear headings and logical flow
- Prefer bulleted lists for non-sequential options or features
- Use semantic, readable URLs and filenames

**Self-check questions:**
- Could an LLM accurately understand and explain this content?
- Can a developer copy and paste these examples and have them run successfully?
- Are all dependencies, versions, and setup steps clearly stated?

### Mintlify formatting conventions

Base docs use [Mintlify](https://mintlify.com) for rendering. Follow these conventions:

**Headings:**
- Start main sections with H2 (`##`)
- Use H3 (`###`) for subsections
- Use H4 (`####`) sparingly for minor subdivisions

**Code blocks:**
```typescript filename="example.ts"
// Use fenced code blocks with language
// Optionally specify filename for context
const example = "like this";
```

**Images:**
```jsx
<Frame>
  <img src="/path/to/image.png" alt="Descriptive text for accessibility" />
</Frame>
```

**Callouts:**
- `<Note>` — General information or clarification
- `<Tip>` — Helpful suggestion or best practice
- `<Warning>` — Important caution or common mistake
- `<Info>` — Additional context or reference
- `<Check>` — Success indicator or verification step

**Procedures:**
```jsx
<Steps>
  <Step title="First step">
    Content for step one
  </Step>
  <Step title="Second step">
    Content for step two
  </Step>
</Steps>
```

**Alternative options:**
```jsx
<Tabs>
  <Tab title="Option A">
    Content for option A
  </Tab>
  <Tab title="Option B">
    Content for option B
  </Tab>
</Tabs>
```

**API documentation:**
```jsx
<ParamField name="parameterName" type="string" required>
  Description of the parameter
</ParamField>

<ResponseField name="fieldName" type="object">
  Description of the response field
</ResponseField>
```

### Code example requirements

All code examples must be:

1. **Complete and runnable**: Include all necessary imports, setup, and configuration
2. **Production-ready**: Show proper error handling, edge cases, and validation
3. **Properly annotated**: Specify language, and include filename when it provides useful context
4. **Verifiable**: Show expected output or include verification steps

**Example:**

```typescript filename="transfer-tokens.ts"
import { encodeFunctionData } from 'viem';

async function transferTokens(to: string, amount: bigint) {
  try {
    const data = encodeFunctionData({
      abi: ERC20_ABI,
      functionName: 'transfer',
      args: [to, amount],
    });
    
    // Send transaction...
    return data;
  } catch (error) {
    console.error('Transfer failed:', error);
    throw error;
  }
}

// Expected output: 0xa9059cbb000000...
```

## Third-party product documentation

> **Warning:** We generally do not accept guides that primarily document third-party products.

**Exceptions require:**
- A clear, specific Base-focused use case
- Deep integration with Base products (Account, App, or Chain)
- Demonstration of unique value to Base developers

Simply deploying a contract on Base or connecting to Base Account/Base App is **not sufficient** for inclusion.

**Alternative:** If your goal is to increase discoverability of your product, request inclusion on the [Base Ecosystem page](https://github.com/base/web?tab=readme-ov-file#updating-the-base-ecosystem-page) instead.

## Pre-submission checklist

Before opening your pull request, verify:

- [ ] Content fits within existing structure (no new top-level sections created)
- [ ] Uses minimal, necessary subsections only
- [ ] Terminology is consistent; abbreviations defined on first use
- [ ] Code examples are complete, tested, and runnable
- [ ] Cross-links to related guides, examples, and references are included
- [ ] Mintlify components and heading hierarchy used correctly
- [ ] All images have descriptive `alt` text and are wrapped in `<Frame>` tags
- [ ] Content is AI-friendly: explicit, well-linked, and easy to follow
- [ ] Local preview shows content rendering correctly
- [ ] No broken links or references to non-existent pages

## Pull request process

### Submitting your PR

1. **Create a pull request** to `https://github.com/base/docs`
2. **Write a clear description** that includes:
   - What you changed and why
   - Links to all pages impacted
   - Any relevant context or background
3. **Request review** from the docs team
4. **Respond to feedback** and iterate on your changes
5. **Merge**: Once approved, your changes will be merged and published automatically

### Review timeline

- **Standard SLA**: 2 weeks for most contributions
- **Priority**: First-come, first-served basis, except for urgent fixes
- **Urgency factors**: Security issues, critical bugs, and time-sensitive updates receive expedited review

### After your PR is merged

Changes are automatically deployed to production once merged. Your contribution will be live on [docs.base.org](https://docs.base.org) within minutes.

## Additional resources

### UI component development

If you're working on custom UI components for the documentation, see `storybook/README.md` for details on running Storybook locally and viewing component documentation.

### Getting help

- **Questions about contributing?** Open a [discussion](https://github.com/base/docs/discussions) in the repository
- **Found a bug?** Open an [issue](https://github.com/base/docs/issues)
- **Want to chat?** Join the Base community on [Discord](https://discord.gg/buildonbase)

---

Thank you for contributing to Base documentation! Your efforts help developers worldwide build better applications on Base.
