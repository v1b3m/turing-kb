# Task Rabbit Data Generation Guide

This guide documents the data generation system used in the Task Rabbit project to create realistic tasker profiles.

## Overview

The data generation system creates thousands of TaskRabbit-style tasker profiles using:
- **Faker.js** for structured data (names, locations, dates, stats)
- **Optional LLM integration** for natural language content (bios, reviews, experience descriptions)
- **Local image pools** for offline-capable avatar and photo references

## Directory Structure

```
src/data/
├── generateData.ts           # Main script - generates all 15 categories
├── generateCategory.ts       # Single category generator
├── redistributeData.ts       # Rebalances skill distribution
├── updateAvatars.ts          # Updates JSON to use local image paths
├── downloadImages.ts         # Downloads remote images (legacy)
├── locations.json            # US states and cities data
├── services.json             # Task categories and subcategories
├── skills.json               # Additional skills data
└── generated/                # Output directory
    ├── handyman.json
    ├── plumbing.json
    └── ... (15 category files)
```

## Core Data Types

### Tasker Object

```typescript
interface Tasker {
  id: string;                          // Sequential ID: "tasker-1", "tasker-2"
  slug: string;                        // URL-friendly: "john-d--123"
  name: { firstname: string; lastname: string };
  about: string;                       // Bio (LLM-generated or fallback)
  avatarUrl: string;                   // Local path to avatar image
  verified: boolean;                   // ~80% true
  location: { city: string; state: string; country: "USA" };
  workLocations: string[];             // 1-3 cities served
  taskerSince: string;                 // ISO date (2015-2024)
  isEliteTasker?: boolean;             // ~15% true
  stats: { rating: number; reviewCount: number; taskCount: number };
  skillIds: string[];                  // Flat list for filtering
  skills: Record<string, Skill>;       // Skills keyed by subcategory ID
  languages: string[];                 // ["English", "Spanish", ...]
  availability: Availability;          // Available days/times
}
```

### Skill Object

```typescript
interface Skill {
  id: string;
  name: string;
  rate: { amount: number; currency: "USD" };
  stats: { rating: number; reviewCount: number; taskCount: number };
  tools?: string[];
  vehicles?: string[];
  limits: { minHours?: number };
  experience: string;                  // LLM-generated description
  photos?: string[];                   // Work photos (40% chance)
  reviews: Review[];                   // 3+ reviews per skill
}
```

### Review Object

```typescript
interface Review {
  avatarUrl: string;
  name: string;                        // "Sarah M."
  rating: number;                      // 4 or 5
  comment: string;
  timestamp: string;                   // ISO date
}
```

## Generation Process

### Phase A: Skeleton Generation

The `generateTaskerSkeleton()` function creates structured data using Faker:

1. **Identity**: Random first/last name, sequential ID, URL slug
2. **Location**: Random US state/city from `locations.json`
3. **Skills**: Primary subcategory (uniform distribution) + 0-3 secondary skills from other categories
4. **Stats**: Aggregated from skills (rating 4.0-5.0, review/task counts)
5. **Attributes**: Verified status (~80%), elite status (~15%), languages, availability

### Phase B: LLM Content Generation

If `LLM_API_KEY` is set, the system calls an LLM to generate:
- **About text**: 2-sentence first-person bio
- **Skill experience**: 1-2 sentences per skill
- **Reviews**: 3 reviews per skill with names and comments

The LLM prompt is built with tasker summaries:
```
[0] John from Los Angeles, CA. Skills: TV Mounting, Plumbing. Elite: yes.
[1] Sarah from Chicago, IL. Skills: House Cleaning. Elite: no.
```

Expected response format:
```json
[
  {
    "index": 0,
    "about": "Hi, I'm John! I've been helping...",
    "skills": [
      {
        "experience": "I specialize in TV mounting...",
        "reviews": [
          {"name": "Sarah M.", "rating": 5, "comment": "Great work!"}
        ]
      }
    ]
  }
]
```

### Fallback Generation

Without an LLM, the system generates template-based content:
```typescript
about: `Hi, I'm ${firstname}! I'm a reliable tasker based in ${city}, ${state}.`
experience: `I have extensive experience with ${skillName}...`
```

Fallback reviews use a pool of generic positive comments.

### Phase C: Merge and Output

LLM results are merged back into tasker skeletons, then written to JSON files.

## Scripts

### Generate All Categories

```bash
# Using fallback text (no API calls)
npx tsx src/data/generateData.ts

# Using LLM for better content
LLM_API_KEY=sk-or-... npx tsx src/data/generateData.ts
```

### Generate Single Category

```bash
# List available categories
npx tsx src/data/generateCategory.ts

# Generate specific category
npx tsx src/data/generateCategory.ts plumbing

# Custom count
TASKERS_COUNT=50 npx tsx src/data/generateCategory.ts cleaning
```

### Redistribute Data

Ensures uniform skill distribution and sequential IDs after regeneration:

```bash
npx tsx src/data/redistributeData.ts
```

This preserves LLM content while reordering skills for balanced coverage.

### Update Avatar Paths

Converts remote URLs to local image paths:

```bash
npx tsx src/data/updateAvatars.ts
```

Assigns random images from local pools:
- 71 tasker avatars
- 100 reviewer avatars
- 50 skill photos

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `LLM_API_KEY` | *(empty)* | API key - if unset, uses fallback text |
| `LLM_BASE_URL` | `https://openrouter.ai/api/v1` | OpenAI-compatible endpoint |
| `LLM_MODEL` | `anthropic/claude-3-haiku-20240307` | Model identifier |
| `TASKERS_PER_CATEGORY` | `150` | Taskers per category (generateData.ts) |
| `TASKERS_COUNT` | `150` | Taskers to generate (generateCategory.ts) |
| `LLM_BATCH_SIZE` | `5` | Taskers per LLM request |
| `LLM_MAX_TOKENS` | `4096` | Max tokens per LLM response |

## LLM Provider Examples

### OpenRouter (Recommended)

```bash
# Claude Haiku
LLM_API_KEY=sk-or-... npx tsx src/data/generateData.ts

# GPT-4o Mini
LLM_API_KEY=sk-or-... LLM_MODEL=openai/gpt-4o-mini npx tsx src/data/generateData.ts

# DeepSeek
LLM_API_KEY=sk-or-... LLM_MODEL=deepseek/deepseek-chat npx tsx src/data/generateData.ts
```

### Direct Provider APIs

```bash
# OpenAI
LLM_API_KEY=sk-... LLM_BASE_URL=https://api.openai.com/v1 LLM_MODEL=gpt-4o-mini npx tsx src/data/generateData.ts

# Groq
LLM_API_KEY=gsk_... LLM_BASE_URL=https://api.groq.com/openai/v1 LLM_MODEL=llama-3.1-8b-instant npx tsx src/data/generateData.ts
```

### Local/Self-Hosted

```bash
# Ollama
LLM_API_KEY=dummy LLM_BASE_URL=http://localhost:11434/v1 LLM_MODEL=llama3 npx tsx src/data/generateData.ts
```

## Static Data Files

### services.json

Contains 15 categories with ~120 total subcategories:

```json
[
  {
    "id": "plumbing",
    "name": "Plumbing",
    "subcategories": [
      { "id": "faucet-repair", "name": "Faucet Repair & Installation" },
      { "id": "toilet-repair", "name": "Toilet Repair" }
    ]
  }
]
```

**Categories**: handyman, mounting, moving, furniture-assembly, cleaning, plumbing, electrical, painting, yard-work, personal-assistant, delivery, organization, event-help, home-improvement, tech-help

### locations.json

Contains 44 US states/DC with ~150 cities:

```json
[
  {
    "state": "California",
    "stateCode": "CA",
    "cities": ["Los Angeles", "San Francisco", "San Diego", ...]
  }
]
```

### Category-Specific Data

Each category has predefined:
- **Tools**: Relevant equipment (e.g., plumbing: "Pipe wrench", "Drain snake")
- **Vehicles**: Transportation options (e.g., "Work Van", "Pickup Truck")
- **Rate ranges**: Min/max hourly rates (e.g., plumbing: $50-$130)

## Image Assets

Images stored in `public/assets/images/`:

```
tasker-avatars/    # 71 profile photos (300x300)
review-avatars/    # 100 reviewer photos (150x150)
photos/
├── thumbnails/    # 50 skill work photos (200x150)
└── large/         # 50 skill work photos (800x600)
```

## Image Generation & Management

Image handling is a two-step process: first download images from remote services to build a local pool, then assign those images to generated tasker data.

### Step 1: Download Images

The `downloadImages.ts` script fetches images from remote placeholder services and saves them locally. This builds the image pool used by the data generation scripts.

**Remote URL sources:**
- **Avatars**: `https://i.pravatar.cc/300?u={uuid}` - Random avatar service
- **Photos**: `https://picsum.photos/seed/{id}/400/300` - Random photo service

**Running the download script:**
```bash
npx tsx src/data/downloadImages.ts
```

**How it works:**

1. Scans all generated JSON files for remote URLs
2. Extracts unique image IDs from URLs
3. Downloads images with retry logic (3 attempts, 30s timeout)
4. Processes downloads in parallel (20 concurrent connections)
5. Saves to `public/assets/images/avatars/` and `public/assets/images/photos/`
6. Updates JSON files with local paths

**Output:**
```
Collecting image URLs...
Found 2250 avatars, 450 photos

Downloading avatars...
  Downloaded 2250/2250

Downloading photos...
  Downloaded 450/450

Updating JSON files with local paths...
  Updated handyman.json
  Updated plumbing.json
  ...

Done!
```

**Image ID Extraction:**

The script extracts unique IDs from URLs to create filenames:

```typescript
function extractId(url: string): string {
  // pravatar: https://i.pravatar.cc/300?u=abc-123
  const pravatarMatch = url.match(/\?u=([a-f0-9-]+)/);
  if (pravatarMatch) return pravatarMatch[1];

  // picsum: https://picsum.photos/seed/123/400/300
  const picsumMatch = url.match(/\/seed\/(\d+)\//);
  if (picsumMatch) return picsumMatch[1];

  // fallback: base64 hash of URL
  return Buffer.from(url).toString("base64").slice(0, 16);
}
```

### Step 2: Assign Images to Data

Once images are downloaded, the data generation and update scripts use the local image pools.

**During generation** (`generateCategory.ts`):

Images are assigned directly from local pools:
```typescript
function getLocalAvatarUrl(): string {
  const num = faker.number.int({ min: 1, max: 71 });
  return `/assets/images/tasker-avatars/avatar-${num}.jpg`;
}
```

**After generation** (`updateAvatars.ts`):

Reassigns random local images to all generated JSON files:
```bash
npx tsx src/data/updateAvatars.ts
```

This is useful when you want to redistribute images or after regenerating data.

### Current Image Pools

```
public/assets/images/
├── tasker-avatars/    # 71 profile photos (avatar-1.jpg to avatar-71.jpg)
├── review-avatars/    # 100 reviewer photos (review-1.jpg to review-100.jpg)
└── photos/
    ├── thumbnails/    # 50 skill work photos (photo-1.jpg to photo-50.jpg)
    └── large/         # 50 skill work photos (photo-1.jpg to photo-50.jpg)
```

### Adding New Images to the Pool

To expand the image pools:

1. **Tasker avatars** - Add images to `public/assets/images/tasker-avatars/`:
   - Naming: `avatar-{N}.jpg` (N = next sequential number)
   - Recommended size: 300x300px
   - Update `TASKER_AVATAR_COUNT` constant in `generateCategory.ts` and `updateAvatars.ts`

2. **Review avatars** - Add to `public/assets/images/review-avatars/`:
   - Naming: `review-{N}.jpg`
   - Recommended size: 150x150px
   - Update `REVIEW_AVATAR_COUNT` constant

3. **Skill photos** - Add to `public/assets/images/photos/thumbnails/` and `photos/large/`:
   - Naming: `photo-{N}.jpg`
   - Thumbnail size: 200x150px
   - Large size: 800x600px
   - Update `PHOTO_COUNT` / `SKILL_PHOTOS` constant

After adding images, run `updateAvatars.ts` to redistribute them across generated data.

## Typical Workflow

### Initial Setup (First Time)

1. **Generate initial data** (creates JSON with remote image URLs):
   ```bash
   npx tsx src/data/generateData.ts
   ```

2. **Download images** (fetches images from remote services to build local pool):
   ```bash
   npx tsx src/data/downloadImages.ts
   ```

3. **Redistribute for uniform coverage** (optional):
   ```bash
   npx tsx src/data/redistributeData.ts
   ```

### Regenerating Data (After Initial Setup)

Once the image pool exists, you can regenerate data that uses local images directly:

1. **Generate all categories**:
   ```bash
   npx tsx src/data/generateData.ts
   ```

2. **Update to use local images**:
   ```bash
   npx tsx src/data/updateAvatars.ts
   ```

### Regenerating a Single Category

```bash
npx tsx src/data/generateCategory.ts plumbing
npx tsx src/data/updateAvatars.ts
```

## Usage in Application

Import generated data:

```typescript
import plumbingTaskers from '@/data/generated/plumbing.json';
import { Tasker } from '@/types';

const taskers = plumbingTaskers as unknown as Tasker[];
```

Helper functions available in `@/types`:

```typescript
import {
  getTaskerDisplayName,  // "John D."
  getTaskerRate,         // First skill's hourly rate
  getTaskerFirstSkill,   // First skill object
  isTaskerElite          // Check elite status
} from '@/types';
```

## Key Implementation Details

### Uniform Skill Distribution

Taskers are assigned primary subcategories via round-robin to ensure even coverage:
```typescript
const subcategoryIndex = i % subcatCount;
const primarySubcategory = category.subcategories[subcategoryIndex];
```

### JSON Parsing Resilience

LLM responses are parsed with fallback handling for truncated JSON:
```typescript
function tryParseJSON(text: string): unknown | null {
  // Strip markdown fences
  // Try direct parse
  // Attempt to recover truncated arrays
}
```

### Batch Processing

LLM calls are batched (default: 5 taskers/request) to balance:
- API efficiency (fewer calls)
- JSON reliability (smaller payloads parse better)
