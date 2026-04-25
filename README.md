# jim

```
    .---.                     
    |   |                     
    '---'.--. __  __   ___    
    .---.|__||  |/  `.'   `.  
    |   |.--.|   .-.  .-.   ' 
    |   ||  ||  |  |  |  |  | 
    |   ||  ||  |  |  |  |  | 
    |   ||  ||  |  |  |  |  | 
    |   ||  ||  |  |  |  |  | 
    |   ||__||__|  |__|  |__| 
 __.'   '                     
|      '                      
|____.'                       
```

<!-- https://patorjk.com/software/taag/#p=display&f=Crazy&t=jim&x=none -->


# What is it

Jim is a **spec-driven SDLC plugin for Claude Code**. It gives you a structured development workflow through namespaced slash commands and specialized agents. You talk to Jim like a person.

```
/jim:spec    → define the work
/jim:plan    → research and break it into tasks
/jim:build   → TDD implementation, one task at a time
```

Jim enforces a simple discipline: think before you code. Every feature, bug fix, or refactor starts with a spec. Every spec gets a plan. Every plan gets built test-first.

Jim can also develop itself — skills and agents for the plugin are specs like any other.

## Commands

| Command | What it does |
|---------|-------------|
| `/jim:spec` | Define a feature, bug, or refactor |
| `/jim:plan` | Research codebase + create atomic task plan |
| `/jim:build` | TDD red-green-refactor, one task at a time |
| `/jim:sec` | Security review of a spec, plan, or arbitrary target |
| `/jim:vision` | Create/update project vision |
| `/jim:arch` | Create/update technical architecture |
| `/jim:roadmap` | Create/update execution roadmap |
| `/jim:backlog` | Consolidate deferred work into `BACKLOG.md` |
| `/jim:debug` | Diagnose failures, produce debug report |
| `/jim:brainstorm` | Freeform ideation and exploratory notes |
| `/jim:config` | Scaffold or update `.jim/config.md` for your project |
| `/jim:meta-skill` | Build a jim plugin skill from spec |
| `/jim:meta-agent` | Build a jim plugin agent from spec |

## Agents

| Agent | Role |
|-------|------|
| `@jim:pm` | Specs, vision, roadmap, brainstorms, backlog |
| `@jim:architect` | Plans, architecture |
| `@jim:researcher` | Codebase and landscape investigation |
| `@jim:security` | Security analysis and threat review |
| `@jim:coder` | TDD builds, debugging |
| `@jim:meta` | Plugin development — builds skills, agents, and config |

## How to install

### From the Marketplace

*(coming soon)*

### From source

1. Clone the repo:
   ```bash
   git clone https://github.com/JamSuite/jim.git
   ```

2. Launch Claude Code in your project with `--plugin-dir` pointing to your local clone:
   ```bash
   claude --plugin-dir /path/to/jim
   ```

That's it — Jim's slash commands and agents are now available in your session.

### Configure for your project (optional)

Jim works zero-config — sensible defaults apply when no config is present. If your project uses different paths or you want to enforce workflow gates, run `/jim:config` to scaffold `.jim/config.md` interactively. Rerun any time to update.

See [`skills/_shared/config-schema.md`](skills/_shared/config-schema.md) for the full schema — paths, spec ID format, workflow gates, validation rules, and the overlay directory.

## How to develop for Jim

See [`WORKFLOW.md`](WORKFLOW.md) for the full SDLC process.

Jim builds itself using its own workflow. Jim's specs live in [`docs/specs/`](docs/specs/).

