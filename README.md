# Tutum Platform - HTML Technical Documentation

This directory contains the complete technical documentation for the Tutum Platform in HTML format.

## Viewing the Documentation

Open `index.html` in your web browser:

```bash
# macOS
open index.html

# Linux
xdg-open index.html

# Or use a local web server
python3 -m http.server 8000
# Then visit: http://localhost:8000
```

## Documentation Structure

- **index.html** - Platform overview and architecture
- **styles.css** - Shared styling for all pages
- Additional component pages (to be added):
  - engine.html - tutum-engine documentation
  - netlib.html - tutum-netlib documentation
  - proto.html - Protocol Buffers API documentation
  - csi-docker.html - Docker CSI driver documentation
  - csi-k8s.html - Kubernetes CSI driver documentation

## Content Overview

The documentation covers:

1. **Platform Architecture** - Complete system design and component interactions
2. **API Design** - Dual API approach (REST + gRPC) with detailed specifications
3. **Component Details** - In-depth information about each platform component
4. **Build Tools** - Technologies, frameworks, and build processes
5. **Security** - Envelope encryption, mTLS, and security best practices
6. **Deployment** - Kubernetes and Docker deployment models
7. **Getting Started** - Quick start guides and next steps

## Key Features Documented

- **tutum-engine**: Core certificate management service with REST/gRPC APIs
- **tutum-netlib**: JDBC-like client library with gRPC implementation
- **proto**: Protocol Buffer definitions and API versioning
- **tutum-csi-driver-k8s**: Kubernetes CSI Driver (v1.5.0) implementation
- **tutum-csi-driver-docker**: Docker Volume Plugin with systemd deployment

## Technology Stack

- **Language**: Go 1.21+
- **Protocols**: REST (HTTP/JSON), gRPC (Protobuf v3)
- **Database**: PostgreSQL 15+
- **Security**: AES-256-GCM envelope encryption, mTLS
- **Container Orchestration**: Kubernetes, Docker Engine
- **Build Tools**: Make, Buf (Protocol Buffers), golangci-lint

## Source Documentation

All content is derived from the following source documents:

- `/TUTUM-PLATFORM.md` - Platform overview
- `/tutum-engine/README.md` - Engine documentation
- `/proto/README.md` - Protocol Buffer specifications
- `/tutum-csi-driver-docker/BUILD.md` - Docker driver build guide
- `/IMPLEMENTATION-COMPLETE.md` - Implementation summary

## Diagram Technology

The architecture diagrams use [Mermaid.js](https://mermaid.js.org/), a JavaScript-based diagramming tool that renders diagrams from text definitions. Benefits:

- **Clean and Readable**: No alignment issues like ASCII art
- **Browser-Rendered**: Scales beautifully on any screen size
- **Interactive**: Modern, professional appearance
- **Maintainable**: Easy to update diagram structure

The diagrams are rendered client-side in the browser, so no additional build steps are required.

## Generating Additional Pages

To add more component-specific pages, create new HTML files following the same structure as `index.html`:

1. Use the same navigation structure
2. Link to `styles.css` for consistent styling
3. Include Mermaid.js script in the `<head>` section for diagrams
4. Follow the semantic HTML5 structure
5. Add appropriate cross-references between pages

## License

MIT License - See main platform LICENSE file.
