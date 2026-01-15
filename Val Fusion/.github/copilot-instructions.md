# VAL FUSION - AI Agent Instructions

## Project Overview
**VAL FUSION** is a Symfony 7.4 corporate website for a French multi-service company group (BTP, security, training, consulting). Built with Docker, TailwindCSS CDN, and custom CSS animations.

## Architecture & Stack
- **Backend**: Symfony 7.4 (PHP 8.2) with Doctrine ORM
- **Frontend**: Twig templates + TailwindCSS (CDN) + custom CSS animations + AOS library
- **Database**: MySQL 8.0
- **Infrastructure**: Docker Compose (nginx, php-fpm, mysql, phpmyadmin)
- **Port Access**: http://localhost:8000 (site), http://localhost:8082 (phpMyAdmin)

## Critical Developer Workflows

### Docker Commands (PowerShell on Windows)
```powershell
# From "Val Fusion" directory, NOT parent folder
docker-compose up -d              # Start containers
docker-compose build              # Rebuild images
docker-compose restart nginx      # Apply frontend changes
docker-compose exec php bash      # Access PHP container

# Clear Symfony cache after CSS/JS/Twig changes
docker-compose exec php php bin/console cache:clear --no-warmup
```

### Makefile Shortcuts
Use `make <command>` for common tasks (see [Makefile](../Makefile) for full list):
- `make install` - Complete setup (build, up, composer, db)
- `make shell` - Access PHP container bash
- `make db-migrate` - Run database migrations

## Project-Specific Conventions

### Routing & Controllers
- **PHP 8+ Attributes**: Routes use `#[Route('/path', name: 'app_name')]` instead of annotations
- **Single Controller**: All pages in `src/Controller/HomeController.php` (simple brochure site pattern)
- **Route Names**: Prefix with `app_` (e.g., `app_val_btp`, `app_contact`)

### Frontend Architecture
**Critical**: This project uses a **hybrid CSS approach**:
1. **TailwindCSS via CDN** - Utility classes in Twig templates
2. **Custom CSS** - `public/css/style.css` (~1000 lines) for complex animations
3. **Inline Styles** - `templates/base.html.twig` has inline `<style>` blocks

**Animation System** (key pattern):
- Classes prefixed `btp-*` for custom animations (e.g., `btp-scroll-reveal`, `btp-fade-in`)
- JavaScript in `public/js/main.js` uses IntersectionObserver to add `.btp-visible` class on scroll
- Always link `style.css` AFTER CDN libraries in base template: 
  ```html
  <link href="unpkg.com/aos@2.3.1/dist/aos.css" rel="stylesheet">
  <link rel="stylesheet" href="{{ asset('css/style.css') }}">
  ```

### Twig Template Structure
```
templates/
├── base.html.twig           # Master layout (head, footer, scripts)
├── home/                    # All page templates
│   ├── index.html.twig      # Homepage
│   ├── val_btp.html.twig    # BTP division page
│   ├── val_concept.html.twig # Consulting division page
│   └── ...
└── partials/
    ├── header.html.twig     # Site navigation
    └── footer.html.twig     # Site footer
```

**Twig Patterns**:
- All pages extend `base.html.twig` with `{% extends 'base.html.twig' %}`
- Override `{% block title %}` and `{% block body %}`
- Use `{{ path('app_route_name') }}` for internal links
- Use `{{ asset('css/file.css') }}` for static assets

### Custom Animation Patterns

**When adding animations to a new page**:
1. Add container class `btp-scroll-reveal` for scroll-triggered elements
2. Use inline `style="animation-delay: 0.2s;"` for sequential reveals
3. Service cards need `btp-service-card` + `btp-card-shine` div inside
4. Gallery items need `btp-gallery-item` class
5. Emoji/icons can use `concept-emoji-bounce` for playful bounce

**Example structure** (from val_btp.html.twig):
```twig
<section class="py-20 bg-white">
  <div class="btp-scroll-reveal">
    <h2 class="btp-heading-reveal">Title</h2>
  </div>
  <div class="btp-service-card">
    <div class="btp-card-shine"></div> {# Shine effect #}
    <h3>Service</h3>
  </div>
</section>
```

## Common Tasks

### Adding a New Page
1. Add route in `src/Controller/HomeController.php`:
   ```php
   #[Route('/new-page', name: 'app_new_page')]
   public function newPage(): Response {
       return $this->render('home/new_page.html.twig');
   }
   ```
2. Create `templates/home/new_page.html.twig` extending `base.html.twig`
3. Add link in `templates/partials/header.html.twig` navigation
4. Clear cache: `docker-compose exec php php bin/console cache:clear`

### Updating Animations/CSS
1. Edit `public/css/style.css` OR `public/js/main.js`
2. **Critical**: Clear cache AND restart nginx:
   ```powershell
   docker-compose exec php php bin/console cache:clear --no-warmup
   docker-compose restart nginx
   ```
3. Hard refresh browser: `Ctrl+F5` (bypass browser cache)

### Database Changes
1. Modify entity in `src/Entity/`
2. Generate migration: `docker-compose exec php php bin/console make:migration`
3. Review migration file in `migrations/`
4. Run migration: `docker-compose exec php php bin/console doctrine:migrations:migrate`

## Integration Points
- **Contact Form**: Uses Symfony Form component (`src/Form/ContactType.php`) + Doctrine ORM
- **Database**: MySQL container `valfusion_database` (credentials in docker-compose.yml)
- **Static Assets**: Served by nginx from `/var/www/html/public`

## Troubleshooting

**Page shows blank/white screen**:
- Likely Twig syntax error or missing asset
- Check PHP logs: `docker-compose logs php --tail=50`
- Verify base.html.twig has `{{ asset('css/style.css') }}` link

**Animations not working**:
- Ensure `style.css` loads after AOS library in base template
- Check browser console (F12) for JS errors
- Verify IntersectionObserver code in `main.js` is initialized

**Docker issues**:
- Always run commands from `Val Fusion/` directory (not parent)
- If containers won't start: `docker-compose down && docker-compose up -d`
- Port conflicts: Check if 8000, 8082, or 3307 are already in use

## Key Files to Reference
- [HomeController.php](../src/Controller/HomeController.php) - All routing logic
- [base.html.twig](../templates/base.html.twig) - Master template structure
- [style.css](../public/css/style.css) - Custom animations (btp-*, concept-* classes)
- [main.js](../public/js/main.js) - Scroll animation triggers
- [docker-compose.yml](../docker-compose.yml) - Infrastructure setup
- [Makefile](../Makefile) - Development shortcuts
