# American Manufacturing 2026

Injection molding factory simulation game celebrating USA's 250th Anniversary.

## Project Structure

```
injection-molding-sim/
├── public/             ← Game files (deploy this folder)
│   ├── index.html      ← Main game
│   ├── images/         ← Game assets (flags, favicons)
│   └── changes/        ← Changelog
├── plans/              ← Design docs & future features
├── LICENSE
└── README.md
```

## Local Development

Simply open `public/index.html` in a browser, or use a local server:

```bash
# .NET
dotnet serve -p 3000 -d public

# Python
python -m http.server 3000 -d public

# Node.js
npx http-server public -p 3000
```

Then visit `http://localhost:3000`

### Debug Mode

Press `` ` `` (backtick) to toggle debug panel for testing:
- Adjust money, parts, revenue
- Change level (1-5)
- Test different game states

## Deployment

### Cloudflare Pages

**Settings:**
- Build output directory: `public`
- Build command: (leave empty)
- Root directory: (leave empty)

The `public/` folder contains only game files. Development docs and plans stay in the root and won't be deployed.

## Credits

Original concept by [Aaron Slodov](https://x.com/aphysicist) and [Atomic Industries](https://x.com/atomic_inc)

American Reindustrialization Edition by Hondo, Hawk and Kijana Woodard of [kaizen.io](https://www.kaizen.io/)
