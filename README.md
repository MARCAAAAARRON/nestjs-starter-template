<div align="center">

# 🚀 NestJS Starter Template

**A minimal, opinionated NestJS starter with global validation baked in — clone it and start building a new resource in minutes.**

[![NestJS](https://img.shields.io/badge/NestJS-E0234E?style=for-the-badge&logo=nestjs&logoColor=white)](https://nestjs.com/)
[![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](./LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=for-the-badge)](CONTRIBUTING.md)

![GitHub stars](https://img.shields.io/github/stars/MARCAAAAARRON/nestjs-starter-template?style=social)
![GitHub forks](https://img.shields.io/github/forks/MARCAAAAARRON/nestjs-starter-template?style=social)
![GitHub last commit](https://img.shields.io/github/last-commit/MARCAAAAARRON/nestjs-starter-template)

</div>

---

## 📖 Table of Contents

- [What's Included](#-whats-included)
- [Setup](#-setup)
- [The Core Pattern](#-the-core-pattern-how-a-request-flows-through-the-app)
- [Adding a New Resource](#-adding-a-new-resource)
- [Full Worked Example](#-full-worked-example-entity--service)
- [Common Mistakes to Avoid](#-common-mistakes-to-avoid)
- [Environment Variables](#-environment-variables)
- [Project Structure](#-project-structure)
- [Contributing](#-contributing)
- [Contributors](#-contributors)
- [License](#-license)

---

## ✨ What's Included
- NestJS base project (Nest CLI default)
- `class-validator` + `class-transformer` installed and wired up
- Global `ValidationPipe` enabled in `main.ts`
- Clean `app.module.ts` with no leftover example resources
- `.env.example` showing required environment variables

---

## ⚙️ Setup

1. Clone this repo
   ```bash
   git clone https://github.com/MARCAAAAARRON/nestjs-starter-template.git
   ```
3. Copy `.env.example` to `.env`
   ```bash
   cp .env.example .env
   ```
4. Fill in real values in `.env`
5. Install dependencies
   ```bash
   pnpm install
   ```
6. Start the dev server
   ```bash
   npm run start:dev
   ```
7. Visit `http://localhost:3000` — you should see "Hello World!"

---

## 🔄 The Core Pattern: How a Request Flows Through the App

Every resource you build in this project follows the same shape. Understanding this flow matters more than memorizing syntax.

```
Client sends a request (e.g. POST /products)
        │
        ▼
┌───────────────────┐
│     Controller      │  "Which route was hit? What HTTP method?
│ (products.controller.ts)   What data came in the body/params?"
└───────────────────┘
        │
        │  passes incoming data through the DTO first
        ▼
┌───────────────────┐
│        DTO           │  "Is this data allowed in? Does it match
│  (dto/create-x.dto.ts)     the required shape and validation rules?"
└───────────────────┘
        │
        │  if valid, Controller calls the Service
        ▼
┌───────────────────┐
│      Service          │  "Here's the actual logic — create,
│ (products.service.ts)      read, update, delete."
└───────────────────┘
        │
        │  reads/writes data shaped like...
        ▼
┌───────────────────┐
│       Entity           │  "This is what a Product actually
│ (entities/product.entity.ts) looks like — its fields and types."
└───────────────────┘
        │
        ▼
Response sent back to the client
```

Everything is tied together by the **Module**, which doesn't contain any logic itself — it just declares "this Controller and this Service belong together":

```
┌─────────────────────────┐
│      ProductsModule        │
│                             │
│  controllers: [ProductsController]
│  providers:   [ProductsService]
└─────────────────────────┘
```

And `AppModule` (the root module) just imports each feature module:

```
AppModule
   └── imports: [ProductsModule, UsersModule, ...]
```

### The Five Pieces, in Plain Terms

| Piece | Question it answers | Example file |
|---|---|---|
| **Entity** | "What does this data look like?" | `entities/product.entity.ts` |
| **DTO** | "What's allowed to come in from the client?" | `dto/create-product.dto.ts` |
| **Controller** | "Which URL/method triggers what?" | `products.controller.ts` |
| **Service** | "What's the actual logic?" | `products.service.ts` |
| **Module** | "How do these pieces get bundled together?" | `products.module.ts` |

**Rule of thumb:** the Controller never contains logic (it just calls the Service), the Service never touches HTTP directly (it just does work and returns data), and a Module never contains logic at all (it's just a manifest).

---

## 🧩 Adding a New Resource

```bash
npx nest generate resource <name>
```

This scaffolds all five pieces above automatically. Then fill in, in this order:

1. **Entity** (`entities/<name>.entity.ts`) — define the fields and their types
2. **DTO** (`dto/create-<name>.dto.ts`) — add `class-validator` decorators for each field (`@IsString()`, `@IsNumber()`, `@IsPositive()`, `@IsIn([...])`, etc.)
3. **Service** (`<name>.service.ts`) — implement `create`, `findAll`, `findOne`, `update`, `remove`
4. **Confirm** `<name>.module.ts` is auto-imported into `app.module.ts` (the CLI usually does this for you — verify it)
5. **Test** every route with a valid request and at least one intentionally invalid request, to confirm your DTO validation actually rejects bad data

### A Note on Fields the Client Shouldn't Control

Some fields (like a `status` or `inStock` flag defaulting to a fixed value) should **not** appear in the `Create` DTO — they belong hardcoded in the Service instead, so a client can never set them directly on creation.

---

## 🛠️ Full Worked Example: Entity + Service

This is a complete, working reference for a `Product` resource — the same shape every future resource in this template will follow. Use this as the template to copy from when building something new, rather than starting from a blank file.

### Entity — `entities/product.entity.ts`

The Entity just describes the *shape* of the data. No logic lives here — it's a plain description of what fields exist and their types.

```typescript
export class Product {
  id!: number;
  name!: string;
  price!: number;
  category!: string;
  inStock!: boolean;
}
```

> Note the `!` after each property name. TypeScript's strict mode normally requires properties to be initialized immediately, but these values get assigned later (by the Service, or by decorators in a real database Entity) — the `!` tells TypeScript "trust me, this will be set before it's used."

### DTO — `dto/create-product.dto.ts`

The DTO describes what's allowed to come **in** from the client. Notice `inStock` is deliberately **not** here — the client never gets to set it directly.

```typescript
import { IsString, IsNotEmpty, IsNumber, IsPositive, IsIn } from 'class-validator';

export class CreateProductDto {
  @IsString()
  @IsNotEmpty()
  name!: string;

  @IsNumber()
  @IsPositive()
  price!: number;

  @IsString()
  @IsNotEmpty()
  @IsIn(['electronics', 'clothing', 'food', 'other'])
  category!: string;
}
```

### Service — `products.service.ts`

The Service is where the real logic lives — every CRUD operation, fully implemented. This is the piece worth studying most closely, since it's where the Entity and DTO actually get used together.

```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { CreateProductDto } from './dto/create-product.dto';
import { UpdateProductDto } from './dto/update-product.dto';
import { Product } from './entities/product.entity';

@Injectable()
export class ProductsService {
  // In-memory storage for now — this array is what a real database
  // table will eventually replace. Everything else stays the same shape.
  private products: Product[] = [];
  private nextId = 1;

  create(dto: CreateProductDto): Product {
    const product: Product = {
      id: this.nextId++,       // server-generated, client never sends this
      name: dto.name,          // comes straight from the validated DTO
      price: dto.price,
      category: dto.category,
      inStock: true,           // hardcoded default — NOT from the client
    };
    this.products.push(product);
    return product;             // always return what was created
  }

  findAll(): Product[] {
    return this.products;
  }

  findOne(id: number): Product {
    const product = this.products.find((p) => p.id === id);
    if (!product) {
      // Use Nest's built-in exceptions, never plain `throw new Error(...)`,
      // so the client gets a proper 404 instead of a generic 500.
      throw new NotFoundException(`Product ${id} not found`);
    }
    return product;
  }

  update(id: number, dto: UpdateProductDto): Product {
    const product = this.findOne(id); // reuse findOne — it already throws if missing
    Object.assign(product, dto);       // merge only the fields the client sent
    return product;
  }

  remove(id: number): void {
    const index = this.products.findIndex((p) => p.id === id);
    if (index === -1) {
      throw new NotFoundException(`Product ${id} not found`);
    }
    this.products.splice(index, 1);
    // void — deleting is an action, not a question, so nothing meaningful to return
  }
}
```

**Things to notice in this Service, in order of importance:**
1. `create()` explicitly hardcodes `inStock: true` — this is the pattern for any field the server should control instead of the client.
2. `findOne()` throws `NotFoundException`, not a plain `Error` — this is what turns into a real `404 Bad Request` HTTP response instead of a generic crash.
3. `update()` reuses `findOne()` rather than duplicating the "find or throw" logic — don't repeat yourself even in a small Service like this.
4. `remove()` returns `void` because deleting doesn't need to hand back a value — it's an action, not a question.
5. Every method that "finds" something returns the actual object; every method that "does" something (`create`, `remove`) either returns the new state or nothing at all.

Once a real database is introduced later, only the *inside* of these methods changes (e.g. `this.products.find(...)` becomes a repository `.findOne(...)` call) — the method signatures, the Controller, and the DTOs all stay exactly the same. This is the entire reason for structuring things this way.

---

## ⚠️ Common Mistakes to Avoid

- **A module importing itself** — a module only lists what it directly owns (`controllers`, `providers`). It never puts its own name in its own `imports` array.
- **Forgetting `ValidationPipe`** — a DTO with decorators does nothing unless `app.useGlobalPipes(new ValidationPipe())` is present in `main.ts` (already done in this template).
- **Using `throw new Error(...)` instead of `NotFoundException`** — plain errors become generic `500`s. Use Nest's built-in exceptions (`NotFoundException`, `BadRequestException`, etc.) so the client gets a proper HTTP status and message.
- **Property initializer errors** — if TypeScript complains a property "has no initializer," add `!` after the name (e.g. `id!: number;`), since decorators/DI assign these values at runtime, not at class definition time.
- **Sending fields the server should control** — don't let the client set fields like `inStock`, `status`, or `createdAt` through the Create DTO; hardcode or auto-generate them in the Service.

---

## 🔑 Environment Variables

See `.env.example` for the full list of variables this project expects. Copy it to `.env` and fill in real values — `.env` is gitignored and should never be committed.

---

## 📁 Project Structure

```
src/
├── app.controller.ts     # default root route ("Hello World")
├── app.service.ts
├── app.module.ts          # root module — imports every feature module
├── main.ts                 # bootstrap + global ValidationPipe
└── <resource>/              # created per-resource via `nest generate resource`
    ├── dto/
    │   ├── create-<resource>.dto.ts
    │   └── update-<resource>.dto.ts
    ├── entities/
    │   └── <resource>.entity.ts
    ├── <resource>.controller.ts
    ├── <resource>.service.ts
    └── <resource>.module.ts
```

---

## 🤝 Contributing

Contributions, issues, and feature suggestions are welcome.

1. Fork the repo
2. Create your feature branch
   ```bash
   git checkout -b feature/amazing-addition
   ```
3. Commit your changes
   ```bash
   git commit -m "Add: amazing addition"
   ```
4. Push to the branch
   ```bash
   git push origin feature/amazing-addition
   ```
5. Open a Pull Request

Please keep additions consistent with the existing pattern (Entity → DTO → Service → Controller → Module) and update the README if you introduce a new convention.

---

## 👥 Contributors

Thanks to everyone who has contributed to this template.

<a href="https://github.com/MARCAAAAARRON/nestjs-starter-template/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=MARCAAAAARRON/nestjs-starter-template" />
</a>

<sub>Built and maintained by [@MARCAAAAARRON](https://github.com/MARCAAAAARRON). Made with the [contrib.rocks](https://contrib.rocks) contributor graph — it updates automatically as people contribute.</sub>

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](./LICENSE) file for details. Free to use, modify, and reuse across any of your future projects.

---

<div align="center">

**If this template saved you setup time, consider giving it a ⭐ on GitHub.**

</div>
