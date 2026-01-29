# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Hallucinations Detector** is a Next.js application that fact-checks LLM-generated content using Exa.ai's search API and Claude 3.7 Sonnet. It extracts claims from user input, searches for verification sources, and provides accuracy assessments with suggested corrections.

## Key Technologies

- **Next.js 14.1.1** with App Router (TypeScript)
- **Exa.ai API** (`exa-js` v1.3.2) - AI-optimized search engine for claim verification
- **Anthropic Claude 3.7 Sonnet** via Vercel AI SDK v3 (`@ai-sdk/anthropic`, `ai`)
- **TailwindCSS** + shadcn/ui components
- **Vercel** hosting with custom basePath `/hallucination-detector`

## Development Commands

```bash
# Install dependencies
npm install

# Run development server (localhost:3000/hallucination-detector)
npm run dev

# Build for production (Vercel handles this automatically)
npm run build

# Production server
npm start

# Lint
npm run lint
```

## Environment Setup

Required environment variables in `.env.local`:

```bash
EXA_API_KEY=your_exa_api_key           # Get from https://dashboard.exa.ai/api-keys
ANTHROPIC_API_KEY=your_anthropic_api_key # Get from https://docs.anthropic.com/en/api/getting-started
```

## Architecture & Data Flow

### Three-Stage Fact-Checking Pipeline

1. **Claim Extraction** (`/api/extractclaims`)
   - Receives user-submitted content
   - Uses Claude 3.7 Sonnet to extract verifiable claims
   - Returns JSON array: `[{claim: string, original_text: string}]`
   - Uses `generateText()` from Vercel AI SDK

2. **Source Retrieval** (`/api/exasearch`)
   - Takes each extracted claim
   - Queries Exa.ai with `searchAndContents()` for 3 sources (`livecrawl: 'always'`)
   - Returns simplified results: `[{text: string, url: string}]`

3. **Claim Verification** (`/api/verifyclaims`)
   - Receives claim + original text + Exa sources
   - Uses Claude 3.7 Sonnet with `generateObject()` and Zod schema
   - Returns structured assessment:
     ```typescript
     {
       claim: string
       assessment: "True" | "False" | "Insufficient Information"
       summary: string
       fixed_original_text: string  // Corrected version if False
       confidence_score: number     // 0-100
     }
     ```

### Client-Side Orchestration

`components/FactChecker.tsx` coordinates the pipeline:
- Calls `extractClaims()` → `exaSearch()` → `verifyClaim()` sequentially
- Uses `Promise.all()` to verify multiple claims in parallel
- Stores results in React state for rendering

### Key Implementation Details

- **basePath Configuration**: All routes use `/hallucination-detector` prefix (configured in `next.config.mjs`)
- **Asset Path Helper**: `getAssetPath()` in `lib/utils.ts` prepends basePath to API routes and assets
- **Middleware Redirect**: `middleware.ts` redirects `exa-hallucination-detector.vercel.app` → `demo.exa.ai/hallucination-detector`
- **API Duration**: Both `/extractclaims` and `/verifyclaims` set `maxDuration = 60` seconds for long LLM operations
- **Error Handling**: All API routes return structured JSON errors with 400/500 status codes

## Component Structure

- `FactChecker.tsx` - Main orchestrator component with form and pipeline logic
- `ClaimsListResult.tsx` - Displays all verified claims with assessments
- `PreviewBox.tsx` - Shows original content with inline annotations
- `PreviewClaimCard.tsx` - Individual claim card with sources
- `ui/LoadingMessages.tsx` - Loading state during verification
- `ui/ShareButtons.tsx` - Social sharing functionality

## Important Constraints

1. **Model Version**: Always use `claude-3-7-sonnet-latest` (hardcoded in routes)
2. **Exa Search Config**: 3 results, `livecrawl: 'always'`, `text: true`, `type: "auto"`
3. **basePath Awareness**: All fetch calls must use `getAssetPath()` for API routes
4. **Zod Schema**: Verification responses must match `factCheckSchema` in `/api/verifyclaims/route.ts`
5. **Parallel Verification**: Client processes multiple claims concurrently but maintains claim order

## Deployment

This app is deployed on Vercel with:
- Custom domain: `demo.exa.ai/hallucination-detector`
- Allowed origins: `["demo.exa.ai"]` for server actions
- Environment variables configured in Vercel dashboard

**Never build locally** - push to GitHub and let Vercel handle deployment.

## Modifying the Pipeline

**To change LLM provider:**
- Update imports in API routes (`app/api/extractclaims/route.ts`, `app/api/verifyclaims/route.ts`)
- Modify `model` parameter in `generateText()`/`generateObject()` calls
- Update environment variables

**To adjust Exa search parameters:**
- Edit `exaSearch()` call in `app/api/exasearch/route.ts`
- Modify `numResults`, `livecrawl`, or search query format

**To change verification schema:**
- Update `factCheckSchema` in `app/api/verifyclaims/route.ts`
- Adjust TypeScript types in `components/FactChecker.tsx` (`FactCheckResponse` interface)

<!-- NEXT-AGENTS-MD-START -->[Next.js Docs Index]|root: ./.next-docs|STOP. What you remember about Next.js is WRONG for this project. Always search docs and read before any task.|If docs missing, run this command first: npx @next/codemod agents-md --output CLAUDE.md<!-- NEXT-AGENTS-MD-END -->
