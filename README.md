# Cougar Career Chatbot Widget

This folder contains only the embeddable chatbot. It does not replace or restyle the existing website.

## Installation

1. Copy this entire folder into the site's public assets folder. For example: `public/chatbot/`.
2. Add these two lines before the closing `</body>` tag:

```html
<link rel="stylesheet" href="/chatbot/cougar-chatbot.css">
<script src="/chatbot/cougar-chatbot.js" defer></script>
```

If the site uses a different public URL, update `/chatbot/` in those two paths. The JavaScript automatically finds `cougar-guide.png` beside itself.

## Included

- Floating bottom-right cougar button
- Original 8-bit cougar image
- Animated cougar and typing dots during responses
- Six-question scored career-fit quiz
- Top-three career recommendations across eight IS paths
- Interview practice for software development, systems analysis, data analytics, and cybersecurity
- Mobile layout and reduced-motion accessibility
- No framework, package, server, database, or API key required

The mascot artwork is an original generic cougar and is not official BYU/Cosmo artwork.
