# 🤖 AI Assistant Context: AWS SAA Study Vault

This file provides context for LLMs and AI agents working within this Obsidian Vault.

## 🎯 Goal
The user is studying for the **AWS Certified Solutions Architect Associate (SAA-C03)** exam. The notes are based on **Adrian Cantrill's** course material.

## 🏗️ Structure & Organization
- **Hierarchical Folders**: Data is organized by AWS Domains (e.g., `01 - Networking`, `02 - Compute`).
- **Standardized Metadata**: Every note should have YAML frontmatter with `tags` and `category`.
- **Callout Heavy**: Use Obsidian callouts (`> [!INFO]`, `> [!WARNING]`) to highlight key exam concepts.
- **Naming Convention**: File names should be descriptive (e.g., `VPC - Virtual Private Cloud.md`).

## ✍️ Writing Style for AI Agents
When editing or creating notes:
1. **Be Concise**: Use bullet points. Avoid flowery language.
2. **Exam Focus**: Highlight "Exam Nuggets" (common traps or specific limits).
3. **Adrian Cantrill Style**: Emphasize "How things work" (The "why" behind the service) rather than just a list of features.
4. **Resiliency & Scope**: Always mention if a service is **Global, Regional, or AZ-specific**.
5. **No Placeholders**: If adding information, use actual technical details.
6. **Use Appropriate Coloring**: Wherever appropriate, use Obsidian's callout colors to highlight important information. Colors to use:
	* rgb(255, 192, 0) - Yellow
	* rgb(255, 0, 0) - Red
	* rgb(0, 176, 240) - Blue
	* rgb(240, 75, 200) - Pink
	* rgb(0, 176, 80) - Green

## 🛠️ Tools Used
- **Obsidian**: Markdown-based knowledge base.
- **Git**: Version control for tracking history and preventing regressions.
- **Plugins**: Dataview (lists), Excalidraw (diagrams), Templater (automation).

## 🛡️ Version Control (Git) Strategy
- **History**: All significant edits should be followed by a commit.
- **No Regression**: AI agents should check `git diff` or history and avoid too many rewrites on the unchanged sections, instead they should focus on the changed sections.
- **Cleanup**: Use `.gitignore` to avoid tracking Obsidian workspace noise.

## 📋 Current Progress
- Foundations: Completed
- Networking: In Progress (VPC/DNS draft)
- Compute: In Progress (EC2/AMI draft)
- Storage: In Progress (S3 draft)
- Deployment: In Progress (CloudWatch/CloudFormation draft)
