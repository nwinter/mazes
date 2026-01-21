# AI Agent Instructions for Mazes Project

## Automated Workflow

When a user says "save prompt to DESIGN_PROMPTS.md":
1. Save the current prompt/context to DESIGN_PROMPTS.md
2. Run tests: `npm test`
3. Run build: `npm run build`
4. If tests and build pass, commit with descriptive message
5. Push to remote

## Development Commands

- `npm run dev` - Start dev server
- `npm test` - Run tests
- `npm run build` - Build for production
- `npm run check` - Type checking

## Project Structure

- `/src/routes/` - SvelteKit routes (single page app)
- `/src/lib/components/` - Svelte components
- `/src/app.css` - Tailwind CSS imports

## Git Workflow

- Main branch: `master`
- Always run tests before committing
- Use descriptive commit messages
