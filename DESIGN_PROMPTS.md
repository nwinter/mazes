# Mazes Design Prompts

Development log documenting prompts and design decisions.

## Initial Setup - 2026-01-20

**Prompt**: Extract maze game from cube repo and deploy to mazes.nickwinter.net

**Changes**:
- Extracted MazeGame component from cube repo feat/maze-game branch
- Created standalone SvelteKit project with Tailwind CSS
- Added AI agent scaffolding (AGENTS.md, CLAUDE.md)
- Deployed to Vercel with custom domain

**Game Features**:
- 25 progressive levels from 5x5 to 25x25
- Star rating system based on time and moves
- Touch controls for mobile
- Keyboard controls (WASD/arrows)
- Progress saved to localStorage
- Confetti celebration on level complete
