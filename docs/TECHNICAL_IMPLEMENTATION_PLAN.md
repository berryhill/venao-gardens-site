# Venao Gardens - Technical Implementation Plan
## Next.js Website Architecture & Development

**Document Version:** 1.0  
**Last Updated:** February 4, 2025  
**Purpose:** Technical specification for website development

---

## Table of Contents
1. [Technical Overview](#technical-overview)
2. [Tech Stack](#tech-stack)
3. [Architecture](#architecture)
4. [Kubernetes Deployment](#kubernetes-deployment)
5. [Site Structure](#site-structure)
6. [SEO & GEO Technical Implementation](#seo--geo-technical-implementation)
7. [Performance Requirements](#performance-requirements)
8. [Development Workflow](#development-workflow)
9. [Implementation Roadmap](#implementation-roadmap)

---

## Technical Overview

### Project Summary
Build a high-performance, SEO-optimized real estate website for Venao Gardens using Next.js, deployed on a self-hosted Kubernetes cluster.

### Key Technical Goals
1. **Performance**: Lighthouse score 90+ on all metrics
2. **SEO Excellence**: Full SSG/ISR support, structured data, semantic HTML
3. **GEO Optimization**: AI-search friendly content structure
4. **Scalability**: Kubernetes-native with auto-scaling
5. **Maintainability**: Headless CMS for content management
6. **Developer Experience**: TypeScript, modern tooling, CI/CD

### Success Metrics
| Metric | Target | Tool |
|--------|--------|------|
| Lighthouse Performance | 90+ | Chrome DevTools |
| Lighthouse SEO | 95+ | Chrome DevTools |
| Core Web Vitals | All Green | Google Search Console |
| Time to First Byte (TTFB) | <200ms | WebPageTest |
| First Contentful Paint (FCP) | <1.8s | Lighthouse |
| Largest Contentful Paint (LCP) | <2.5s | Lighthouse |
| Cumulative Layout Shift (CLS) | <0.1 | Lighthouse |

---

## Tech Stack

### Frontend Framework
```
Next.js 14+ (App Router)
├── React 18
├── TypeScript 5.x
├── Server Components (default)
└── Client Components (as needed)
```

**Why Next.js 14+:**
- App Router for modern React patterns
- Built-in image optimization
- Automatic code splitting
- SSG/SSR/ISR support for SEO
- Server Components for performance
- Excellent TypeScript support

### Styling
```
Tailwind CSS 3.4+
├── Custom design system tokens
├── Component variants via cva/class-variance-authority
└── Framer Motion (animations)
```

### Content Management
```
Sanity.io (Headless CMS)
├── Real-time collaboration
├── Structured content modeling
├── Image CDN with transformations
├── GROQ query language
└── TypeScript schema generation
```

**Why Sanity:**
- Generous free tier (100k API requests/month)
- Excellent image pipeline with CDN
- Real-time preview capabilities
- Flexible content modeling
- Strong TypeScript support

### Form Handling
```
React Hook Form + Zod
├── Type-safe form validation
├── Minimal re-renders
└── Server Actions for submission
```

### Analytics & Monitoring
```
├── Google Analytics 4 (marketing)
├── Plausible (privacy-friendly alternative)
├── Vercel Analytics (if using Vercel)
└── Sentry (error tracking)
```

### Full Dependency List
```json
{
  "dependencies": {
    "next": "^14.2.0",
    "react": "^18.3.0",
    "react-dom": "^18.3.0",
    "@sanity/client": "^6.x",
    "@sanity/image-url": "^1.x",
    "next-sanity": "^9.x",
    "framer-motion": "^11.x",
    "react-hook-form": "^7.x",
    "zod": "^3.x",
    "@hookform/resolvers": "^3.x",
    "lucide-react": "^0.x",
    "clsx": "^2.x",
    "tailwind-merge": "^2.x"
  },
  "devDependencies": {
    "typescript": "^5.x",
    "@types/node": "^20.x",
    "@types/react": "^18.x",
    "tailwindcss": "^3.4.x",
    "postcss": "^8.x",
    "autoprefixer": "^10.x",
    "eslint": "^8.x",
    "eslint-config-next": "^14.x",
    "prettier": "^3.x",
    "prettier-plugin-tailwindcss": "^0.6.x"
  }
}
```

---

## Architecture

### Application Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENT BROWSER                           │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    CLOUDFLARE (Optional CDN)                     │
│  - Edge caching for static assets                               │
│  - DDoS protection                                              │
│  - SSL termination (alternative)                                │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     KUBERNETES CLUSTER                           │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │              Ingress Controller (nginx)                    │ │
│  │  - SSL via cert-manager                                    │ │
│  │  - Load balancing                                          │ │
│  └───────────────────────────────────────────────────────────┘ │
│                              │                                   │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │           Next.js Application (2-5 replicas)               │ │
│  │  ┌─────────────────────────────────────────────────────┐  │ │
│  │  │  Server Components (RSC)                            │  │ │
│  │  │  - Fetch from Sanity at build/request time          │  │ │
│  │  │  - Generate static pages (SSG)                      │  │ │
│  │  │  - Incremental regeneration (ISR)                   │  │ │
│  │  └─────────────────────────────────────────────────────┘  │ │
│  │  ┌─────────────────────────────────────────────────────┐  │ │
│  │  │  API Routes                                         │  │ │
│  │  │  - /api/health (k8s probes)                         │  │ │
│  │  │  - /api/contact (form submission)                   │  │ │
│  │  │  - /api/revalidate (ISR webhook)                    │  │ │
│  │  └─────────────────────────────────────────────────────┘  │ │
│  └───────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┴───────────────┐
              ▼                               ▼
┌─────────────────────────┐     ┌─────────────────────────┐
│       SANITY.IO         │     │     EXTERNAL APIs       │
│  - Content storage      │     │  - HubSpot CRM          │
│  - Media CDN            │     │  - SendGrid/Resend      │
│  - Real-time updates    │     │  - Google Maps          │
└─────────────────────────┘     └─────────────────────────┘
```

### Rendering Strategy

| Page Type | Rendering | Revalidation | Rationale |
|-----------|-----------|--------------|-----------|
| Homepage | SSG + ISR | 1 hour | Mostly static, occasional updates |
| /the-vision | SSG | Manual | Rarely changes |
| /location | SSG | Manual | Static content |
| /lots | SSG + ISR | 15 minutes | Lot availability changes |
| /lots/[slug] | SSG + ISR | 15 minutes | Dynamic lot data |
| /investment | SSG | Manual | Static investment info |
| /about | SSG | Manual | Static company info |
| /contact | SSG | None | Static form |
| /blog | SSG + ISR | 1 hour | New posts periodically |
| /blog/[slug] | SSG + ISR | 1 hour | Post updates rare |

### Project Structure
```
venao-gardens-site/
├── .github/
│   └── workflows/
│       └── deploy.yml           # CI/CD pipeline
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── hpa.yaml
│   └── secrets.yaml.example
├── public/
│   ├── images/
│   ├── fonts/
│   ├── robots.txt
│   └── sitemap.xml              # Generated
├── src/
│   ├── app/
│   │   ├── (marketing)/         # Marketing pages group
│   │   │   ├── page.tsx         # Homepage
│   │   │   ├── the-vision/
│   │   │   ├── location/
│   │   │   ├── lots/
│   │   │   │   ├── page.tsx
│   │   │   │   └── [slug]/
│   │   │   ├── investment/
│   │   │   ├── about/
│   │   │   └── contact/
│   │   ├── blog/
│   │   │   ├── page.tsx
│   │   │   └── [slug]/
│   │   ├── api/
│   │   │   ├── health/
│   │   │   ├── contact/
│   │   │   └── revalidate/
│   │   ├── layout.tsx
│   │   ├── not-found.tsx
│   │   └── sitemap.ts
│   ├── components/
│   │   ├── ui/                  # Base UI components
│   │   │   ├── button.tsx
│   │   │   ├── input.tsx
│   │   │   ├── card.tsx
│   │   │   └── ...
│   │   ├── layout/              # Layout components
│   │   │   ├── header.tsx
│   │   │   ├── footer.tsx
│   │   │   └── navigation.tsx
│   │   ├── sections/            # Page sections
│   │   │   ├── hero.tsx
│   │   │   ├── value-props.tsx
│   │   │   ├── lot-grid.tsx
│   │   │   └── ...
│   │   └── forms/               # Form components
│   │       ├── contact-form.tsx
│   │       └── lead-capture.tsx
│   ├── lib/
│   │   ├── sanity/
│   │   │   ├── client.ts
│   │   │   ├── queries.ts
│   │   │   └── types.ts
│   │   ├── utils.ts
│   │   └── constants.ts
│   ├── styles/
│   │   └── globals.css
│   └── types/
│       └── index.ts
├── sanity/
│   ├── schemas/
│   │   ├── lot.ts
│   │   ├── page.ts
│   │   ├── post.ts
│   │   └── settings.ts
│   └── sanity.config.ts
├── Dockerfile
├── docker-compose.yml           # Local development
├── next.config.js
├── tailwind.config.ts
├── tsconfig.json
├── package.json
└── README.md
```

---

## Kubernetes Deployment

### Cluster Requirements
- **Ingress Controller**: nginx-ingress or traefik
- **SSL**: cert-manager with Let's Encrypt
- **Container Registry**: Docker Hub, GHCR, or private registry
- **Monitoring**: Prometheus + Grafana (recommended)

### Docker Configuration

```dockerfile
# Dockerfile
FROM node:20-alpine AS base

# Dependencies stage
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app

COPY package.json package-lock.json* ./
RUN npm ci --only=production

# Builder stage
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

ENV NEXT_TELEMETRY_DISABLED 1
ENV NODE_ENV production

# Build arguments for environment variables
ARG NEXT_PUBLIC_SANITY_PROJECT_ID
ARG NEXT_PUBLIC_SANITY_DATASET

RUN npm run build

# Runner stage
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public

RUN mkdir .next
RUN chown nextjs:nodejs .next

COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000
ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/api/health || exit 1

CMD ["node", "server.js"]
```

### Next.js Configuration for Standalone
```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'standalone',
  
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'cdn.sanity.io',
      },
    ],
  },
  
  // Enable experimental features as needed
  experimental: {
    // serverActions: true, // Enabled by default in Next.js 14
  },
  
  // Headers for security
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          {
            key: 'X-DNS-Prefetch-Control',
            value: 'on',
          },
          {
            key: 'X-Frame-Options',
            value: 'SAMEORIGIN',
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff',
          },
        ],
      },
    ];
  },
};

module.exports = nextConfig;
```

### Kubernetes Manifests

```yaml
# k8s/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: venao-gardens
---
# k8s/secrets.yaml (example - use sealed-secrets or external-secrets in production)
apiVersion: v1
kind: Secret
metadata:
  name: venao-gardens-secrets
  namespace: venao-gardens
type: Opaque
stringData:
  SANITY_PROJECT_ID: "your-project-id"
  SANITY_DATASET: "production"
  SANITY_API_TOKEN: "your-api-token"
  HUBSPOT_API_KEY: "your-hubspot-key"
  SENDGRID_API_KEY: "your-sendgrid-key"
---
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: venao-gardens-config
  namespace: venao-gardens
data:
  NODE_ENV: "production"
  NEXT_PUBLIC_SITE_URL: "https://venaogardens.com"
---
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: venao-gardens-web
  namespace: venao-gardens
  labels:
    app: venao-gardens-web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: venao-gardens-web
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: venao-gardens-web
    spec:
      containers:
      - name: nextjs
        image: your-registry/venao-gardens:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 3000
          name: http
        resources:
          requests:
            memory: "256Mi"
            cpu: "200m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        envFrom:
        - configMapRef:
            name: venao-gardens-config
        - secretRef:
            name: venao-gardens-secrets
        livenessProbe:
          httpGet:
            path: /api/health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /api/health
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
---
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: venao-gardens-web
  namespace: venao-gardens
spec:
  selector:
    app: venao-gardens-web
  ports:
  - port: 80
    targetPort: 3000
    name: http
  type: ClusterIP
---
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: venao-gardens-ingress
  namespace: venao-gardens
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
    # Redirect www to non-www
    nginx.ingress.kubernetes.io/from-to-www-redirect: "true"
spec:
  tls:
  - hosts:
    - venaogardens.com
    - www.venaogardens.com
    secretName: venao-gardens-tls
  rules:
  - host: venaogardens.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: venao-gardens-web
            port:
              number: 80
  - host: www.venaogardens.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: venao-gardens-web
            port:
              number: 80
---
# k8s/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: venao-gardens-hpa
  namespace: venao-gardens
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: venao-gardens-web
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Pods
        value: 1
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Pods
        value: 2
        periodSeconds: 60
```

### Health Check Endpoint
```typescript
// src/app/api/health/route.ts
import { NextResponse } from 'next/server';

export async function GET() {
  return NextResponse.json(
    {
      status: 'healthy',
      timestamp: new Date().toISOString(),
      version: process.env.npm_package_version || '1.0.0',
    },
    { status: 200 }
  );
}
```

---

## Site Structure

### Page Components Overview

| Route | Component | Data Source | Priority |
|-------|-----------|-------------|----------|
| `/` | HomePage | Sanity (home settings, featured lots) | P0 |
| `/the-vision` | VisionPage | Sanity (vision content) | P0 |
| `/location` | LocationPage | Sanity (location content) | P0 |
| `/lots` | LotsPage | Sanity (all lots) | P0 |
| `/lots/[slug]` | LotDetailPage | Sanity (single lot) | P1 |
| `/investment` | InvestmentPage | Sanity (investment content) | P1 |
| `/about` | AboutPage | Sanity (about content) | P1 |
| `/contact` | ContactPage | Static + Form | P0 |
| `/blog` | BlogPage | Sanity (posts) | P2 |
| `/blog/[slug]` | PostPage | Sanity (single post) | P2 |

### Sanity Schema Models

```typescript
// sanity/schemas/lot.ts
export default {
  name: 'lot',
  title: 'Lot',
  type: 'document',
  fields: [
    {
      name: 'name',
      title: 'Lot Name',
      type: 'string',
      validation: (Rule: any) => Rule.required(),
    },
    {
      name: 'slug',
      title: 'Slug',
      type: 'slug',
      options: { source: 'name' },
      validation: (Rule: any) => Rule.required(),
    },
    {
      name: 'status',
      title: 'Status',
      type: 'string',
      options: {
        list: [
          { title: 'Available', value: 'available' },
          { title: 'Reserved', value: 'reserved' },
          { title: 'Sold', value: 'sold' },
        ],
      },
      initialValue: 'available',
    },
    {
      name: 'price',
      title: 'Price (USD)',
      type: 'number',
    },
    {
      name: 'size',
      title: 'Size (m²)',
      type: 'number',
    },
    {
      name: 'viewType',
      title: 'View Type',
      type: 'string',
      options: {
        list: [
          { title: 'Ocean View', value: 'ocean' },
          { title: 'Garden View', value: 'garden' },
          { title: 'Mountain View', value: 'mountain' },
        ],
      },
    },
    {
      name: 'features',
      title: 'Features',
      type: 'array',
      of: [{ type: 'string' }],
    },
    {
      name: 'images',
      title: 'Images',
      type: 'array',
      of: [{ type: 'image', options: { hotspot: true } }],
    },
    {
      name: 'description',
      title: 'Description',
      type: 'blockContent',
    },
    {
      name: 'mapPosition',
      title: 'Map Position',
      type: 'object',
      fields: [
        { name: 'x', type: 'number', title: 'X Position (%)' },
        { name: 'y', type: 'number', title: 'Y Position (%)' },
      ],
    },
  ],
  preview: {
    select: {
      title: 'name',
      status: 'status',
      price: 'price',
    },
    prepare({ title, status, price }: any) {
      return {
        title,
        subtitle: `${status} - $${price?.toLocaleString() || 'TBD'}`,
      };
    },
  },
};
```

---

## SEO & GEO Technical Implementation

### Metadata Configuration

```typescript
// src/app/layout.tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  metadataBase: new URL('https://venaogardens.com'),
  title: {
    default: 'Venao Gardens | Premier Neighborhood in Playa Venao, Panama',
    template: '%s | Venao Gardens',
  },
  description:
    'Discover Venao Gardens, the premier residential community in Playa Venao, Panama. Premium lots, world-class beach lifestyle, and exceptional investment opportunity.',
  keywords: [
    'Playa Venao real estate',
    'Panama land investment',
    'beach property Panama',
    'Venao Gardens',
    'Azuero Peninsula property',
  ],
  authors: [{ name: 'Venao Gardens Development' }],
  creator: 'Venao Gardens',
  publisher: 'Venao Gardens',
  formatDetection: {
    email: false,
    address: false,
    telephone: false,
  },
  openGraph: {
    type: 'website',
    locale: 'en_US',
    url: 'https://venaogardens.com',
    siteName: 'Venao Gardens',
    title: 'Venao Gardens | Premier Neighborhood in Playa Venao, Panama',
    description:
      'Premium lots in Panama\'s most exciting beach destination. Invest in your paradise today.',
    images: [
      {
        url: '/og-image.jpg',
        width: 1200,
        height: 630,
        alt: 'Venao Gardens - Premier Neighborhood in Playa Venao',
      },
    ],
  },
  twitter: {
    card: 'summary_large_image',
    title: 'Venao Gardens | Premier Neighborhood in Playa Venao',
    description: 'Premium lots in Panama\'s most exciting beach destination.',
    images: ['/og-image.jpg'],
  },
  robots: {
    index: true,
    follow: true,
    googleBot: {
      index: true,
      follow: true,
      'max-video-preview': -1,
      'max-image-preview': 'large',
      'max-snippet': -1,
    },
  },
  verification: {
    google: 'your-google-verification-code',
  },
};
```

### Structured Data Component

```typescript
// src/components/seo/json-ld.tsx
interface JsonLdProps {
  data: Record<string, any>;
}

export function JsonLd({ data }: JsonLdProps) {
  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(data) }}
    />
  );
}

// Organization schema
export const organizationSchema = {
  '@context': 'https://schema.org',
  '@type': 'RealEstateAgent',
  name: 'Venao Gardens',
  description: 'Premier residential development in Playa Venao, Panama',
  url: 'https://venaogardens.com',
  logo: 'https://venaogardens.com/logo.png',
  image: 'https://venaogardens.com/images/development.jpg',
  address: {
    '@type': 'PostalAddress',
    addressLocality: 'Playa Venao',
    addressRegion: 'Los Santos Province',
    addressCountry: 'PA',
  },
  geo: {
    '@type': 'GeoCoordinates',
    latitude: '7.4278',
    longitude: '-80.1633',
  },
  areaServed: {
    '@type': 'Place',
    name: 'Playa Venao, Panama',
  },
  sameAs: [
    'https://www.facebook.com/venaogardens',
    'https://www.instagram.com/venaogardens',
  ],
};

// Lot/Product schema generator
export function generateLotSchema(lot: any) {
  return {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name: lot.name,
    description: lot.description,
    image: lot.images?.[0]?.url,
    brand: {
      '@type': 'Brand',
      name: 'Venao Gardens',
    },
    offers: {
      '@type': 'Offer',
      price: lot.price,
      priceCurrency: 'USD',
      availability:
        lot.status === 'available'
          ? 'https://schema.org/InStock'
          : 'https://schema.org/SoldOut',
      seller: {
        '@type': 'RealEstateAgent',
        name: 'Venao Gardens',
      },
    },
    additionalProperty: [
      {
        '@type': 'PropertyValue',
        name: 'Lot Size',
        value: `${lot.size} m²`,
      },
      {
        '@type': 'PropertyValue',
        name: 'View Type',
        value: lot.viewType,
      },
    ],
  };
}

// FAQ schema generator
export function generateFaqSchema(faqs: Array<{ question: string; answer: string }>) {
  return {
    '@context': 'https://schema.org',
    '@type': 'FAQPage',
    mainEntity: faqs.map((faq) => ({
      '@type': 'Question',
      name: faq.question,
      acceptedAnswer: {
        '@type': 'Answer',
        text: faq.answer,
      },
    })),
  };
}
```

### Dynamic Sitemap Generation

```typescript
// src/app/sitemap.ts
import { MetadataRoute } from 'next';
import { sanityClient } from '@/lib/sanity/client';

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const baseUrl = 'https://venaogardens.com';

  // Static pages
  const staticPages = [
    { url: baseUrl, lastModified: new Date(), changeFrequency: 'weekly' as const, priority: 1 },
    { url: `${baseUrl}/the-vision`, lastModified: new Date(), changeFrequency: 'monthly' as const, priority: 0.8 },
    { url: `${baseUrl}/location`, lastModified: new Date(), changeFrequency: 'monthly' as const, priority: 0.8 },
    { url: `${baseUrl}/lots`, lastModified: new Date(), changeFrequency: 'daily' as const, priority: 0.9 },
    { url: `${baseUrl}/investment`, lastModified: new Date(), changeFrequency: 'monthly' as const, priority: 0.7 },
    { url: `${baseUrl}/about`, lastModified: new Date(), changeFrequency: 'monthly' as const, priority: 0.6 },
    { url: `${baseUrl}/contact`, lastModified: new Date(), changeFrequency: 'yearly' as const, priority: 0.5 },
  ];

  // Dynamic lot pages
  const lots = await sanityClient.fetch<Array<{ slug: string; _updatedAt: string }>>(
    `*[_type == "lot"]{ "slug": slug.current, _updatedAt }`
  );

  const lotPages = lots.map((lot) => ({
    url: `${baseUrl}/lots/${lot.slug}`,
    lastModified: new Date(lot._updatedAt),
    changeFrequency: 'weekly' as const,
    priority: 0.7,
  }));

  // Dynamic blog pages (if applicable)
  const posts = await sanityClient.fetch<Array<{ slug: string; _updatedAt: string }>>(
    `*[_type == "post"]{ "slug": slug.current, _updatedAt }`
  );

  const blogPages = posts.map((post) => ({
    url: `${baseUrl}/blog/${post.slug}`,
    lastModified: new Date(post._updatedAt),
    changeFrequency: 'monthly' as const,
    priority: 0.5,
  }));

  return [...staticPages, ...lotPages, ...blogPages];
}
```

### Robots.txt

```typescript
// src/app/robots.ts
import { MetadataRoute } from 'next';

export default function robots(): MetadataRoute.Robots {
  return {
    rules: {
      userAgent: '*',
      allow: '/',
      disallow: ['/api/', '/admin/'],
    },
    sitemap: 'https://venaogardens.com/sitemap.xml',
  };
}
```

---

## Performance Requirements

### Image Optimization

```typescript
// src/components/ui/optimized-image.tsx
import Image from 'next/image';
import { urlForImage } from '@/lib/sanity/image';

interface OptimizedImageProps {
  image: any; // Sanity image reference
  alt: string;
  priority?: boolean;
  className?: string;
  sizes?: string;
}

export function OptimizedImage({
  image,
  alt,
  priority = false,
  className,
  sizes = '(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw',
}: OptimizedImageProps) {
  const imageUrl = urlForImage(image).url();

  return (
    <Image
      src={imageUrl}
      alt={alt}
      fill
      priority={priority}
      className={className}
      sizes={sizes}
      placeholder="blur"
      blurDataURL={urlForImage(image).width(20).blur(50).url()}
    />
  );
}
```

### Loading States

```typescript
// src/components/ui/skeleton.tsx
import { cn } from '@/lib/utils';

export function Skeleton({
  className,
  ...props
}: React.HTMLAttributes<HTMLDivElement>) {
  return (
    <div
      className={cn('animate-pulse rounded-md bg-muted', className)}
      {...props}
    />
  );
}

// Usage in lot grid
export function LotGridSkeleton() {
  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
      {Array.from({ length: 6 }).map((_, i) => (
        <div key={i} className="space-y-4">
          <Skeleton className="h-48 w-full" />
          <Skeleton className="h-4 w-3/4" />
          <Skeleton className="h-4 w-1/2" />
        </div>
      ))}
    </div>
  );
}
```

---

## Development Workflow

### CI/CD Pipeline

```yaml
# .github/workflows/deploy.yml
name: Build and Deploy

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  lint-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Lint
        run: npm run lint
      
      - name: Type check
        run: npm run type-check
      
      - name: Build
        run: npm run build
        env:
          NEXT_PUBLIC_SANITY_PROJECT_ID: ${{ secrets.SANITY_PROJECT_ID }}
          NEXT_PUBLIC_SANITY_DATASET: ${{ secrets.SANITY_DATASET }}

  build-and-push:
    needs: lint-and-test
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    permissions:
      contents: read
      packages: write
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=sha,prefix=
            type=raw,value=latest
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          build-args: |
            NEXT_PUBLIC_SANITY_PROJECT_ID=${{ secrets.SANITY_PROJECT_ID }}
            NEXT_PUBLIC_SANITY_DATASET=${{ secrets.SANITY_DATASET }}

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up kubectl
        uses: azure/setup-kubectl@v3
      
      - name: Configure kubectl
        run: |
          mkdir -p ~/.kube
          echo "${{ secrets.KUBECONFIG }}" | base64 -d > ~/.kube/config
      
      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/venao-gardens-web \
            nextjs=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} \
            -n venao-gardens
          kubectl rollout status deployment/venao-gardens-web -n venao-gardens
```

### Local Development

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
      target: deps
    volumes:
      - .:/app
      - /app/node_modules
    ports:
      - '3000:3000'
    environment:
      - NODE_ENV=development
      - NEXT_PUBLIC_SANITY_PROJECT_ID=${SANITY_PROJECT_ID}
      - NEXT_PUBLIC_SANITY_DATASET=${SANITY_DATASET}
    command: npm run dev
```

---

## Implementation Roadmap

### Phase 1: Foundation (Week 1-2)

**Week 1: Project Setup**
- [ ] Initialize Next.js project with TypeScript
- [ ] Configure Tailwind CSS with custom theme
- [ ] Set up Sanity project and schemas
- [ ] Configure ESLint, Prettier, Husky
- [ ] Set up Git repository and branch protection
- [ ] Create Docker configuration
- [ ] Set up Kubernetes manifests

**Week 2: Core Components**
- [ ] Build design system (buttons, inputs, cards)
- [ ] Create layout components (header, footer, nav)
- [ ] Implement responsive navigation
- [ ] Build hero section component
- [ ] Create contact form with validation
- [ ] Set up form submission API route

### Phase 2: Pages (Week 3-4)

**Week 3: Primary Pages**
- [ ] Homepage implementation
- [ ] The Vision page
- [ ] Location page
- [ ] Contact page
- [ ] Connect Sanity content

**Week 4: Lots Feature**
- [ ] Lots listing page
- [ ] Interactive site map component
- [ ] Individual lot detail page
- [ ] Filtering and sorting
- [ ] ISR configuration

### Phase 3: Launch Prep (Week 5-6)

**Week 5: Optimization**
- [ ] SEO metadata implementation
- [ ] Structured data (JSON-LD)
- [ ] Image optimization audit
- [ ] Performance testing
- [ ] Accessibility audit
- [ ] CRM integration (HubSpot)

**Week 6: Deploy**
- [ ] CI/CD pipeline setup
- [ ] Staging deployment
- [ ] Content population
- [ ] QA testing
- [ ] Production deployment
- [ ] DNS configuration
- [ ] Monitoring setup

### Milestone Checklist

| Milestone | Target | Verification |
|-----------|--------|--------------|
| Dev environment ready | End Week 1 | Can run `npm run dev` |
| Design system complete | End Week 2 | Component library functional |
| Core pages in staging | End Week 3 | Accessible on staging URL |
| Full site in staging | End Week 4 | All pages functional |
| Performance optimized | End Week 5 | Lighthouse 90+ |
| Production launch | End Week 6 | Live on venaogardens.com |

---

## Appendix

### Environment Variables

```bash
# .env.example

# Sanity
NEXT_PUBLIC_SANITY_PROJECT_ID=
NEXT_PUBLIC_SANITY_DATASET=production
SANITY_API_TOKEN=

# Site
NEXT_PUBLIC_SITE_URL=https://venaogardens.com

# Analytics
NEXT_PUBLIC_GA_ID=
NEXT_PUBLIC_PLAUSIBLE_DOMAIN=

# Forms/CRM
HUBSPOT_API_KEY=
SENDGRID_API_KEY=

# Maps
NEXT_PUBLIC_MAPBOX_TOKEN=
```

### Useful Commands

```bash
# Development
npm run dev          # Start development server
npm run build        # Build for production
npm run start        # Start production server
npm run lint         # Run ESLint
npm run type-check   # Run TypeScript check

# Sanity
npm run sanity:dev   # Start Sanity Studio
npm run sanity:deploy # Deploy Sanity Studio

# Docker
docker build -t venao-gardens .
docker run -p 3000:3000 venao-gardens

# Kubernetes
kubectl apply -f k8s/
kubectl get pods -n venao-gardens
kubectl logs -f deployment/venao-gardens-web -n venao-gardens
```

---

*Technical Implementation Plan - Venao Gardens*  
*Ready for development phase*
