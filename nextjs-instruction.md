/*
 VS Copilot Instructions & Best Practices: Next.js, TypeScript, Supabase

 --- GENERAL ---

 1.  **Naming Conventions:**
     -   Components: PascalCase (`UserProfile.tsx`)
     -   Variables/Functions: camelCase (`getUserData`, `isLoading`)
     -   Constants/Enums: SCREAMING_SNAKE_CASE (`MAX_RETRIES`, `Status.PENDING`)
     -   Types/Interfaces: PascalCase (`UserData`, `IAuthResponse`)
     -   Files/Folders: kebab-case (`user-profile`, `lib/utils`) or camelCase for components/classes. Be consistent.

 2.  **File Structure (App Router):**
     -   `/app`: Main routing structure. Group routes logically.
     -   `/components`: Shared React components (UI, layout). Consider subfolders (`/components/ui`, `/components/auth`).
     -   `/lib`: Shared functions, utilities, Supabase client setup (`/lib/supabaseClient.ts`, `/lib/utils.ts`).
     -   `/types`: Shared TypeScript type definitions (`/types/index.ts`, `/types/supabase.ts`).
     -   `/hooks`: Custom React hooks.
     -   `/styles`: Global styles, Tailwind config (`globals.css`).

 3.  **Environment Variables:**
     -   Use `.env.local` for secrets and local configuration. **NEVER** commit `.env.local`.
     -   Prefix client-side accessible variables with `NEXT_PUBLIC_` (e.g., `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`).
     -   Server-side only variables (e.g., `SUPABASE_SERVICE_ROLE_KEY`) should NOT have the prefix and only be accessed in server environments (Server Components, Route Handlers, Server Actions).

 --- NEXT.JS (APP ROUTER) ---

 4.  **Component Types:**
     -   **Default to Server Components:** Use for data fetching, accessing backend resources directly, and components without interactivity. Do NOT use React hooks like `useState`, `useEffect`, or browser APIs.
     -   **Use Client Components (`'use client'`) sparingly:** Only when necessary for interactivity (event handlers), state (`useState`), lifecycle effects (`useEffect`), or browser-specific APIs. Keep them small and push state/interactivity down the tree.

 5.  **Data Fetching:**
     -   **Server Components:** Fetch data directly using `async/await` with `fetch` or your Supabase client. Leverage Next.js caching and revalidation options.
     -   **Client Components:**
         -   Fetch data via Route Handlers (`/app/api/.../route.ts`) using libraries like `SWR` or `React Query` (or `fetch`).
         -   Alternatively, pass data fetched in a parent Server Component down as props.
         -   For mutations, strongly prefer Server Actions or Route Handlers.

 6.  **API Routes & Mutations:**
     -   **Prefer Server Actions:** For mutations triggered from Client Components. They simplify data mutation logic, run on the server, and integrate well with forms and `useOptimistic`.
     -   **Use Route Handlers (`/app/api/.../route.ts`)** for creating traditional API endpoints accessed by clients or third parties. Ensure proper request/response typing.

 7.  **Routing & Layouts:**
     -   Utilize `layout.tsx` for shared UI across segments.
     -   Use `template.tsx` if you need unique instances per route without shared state.
     -   Use `loading.tsx` for route segment loading state (integrates with Suspense).
     -   Use `error.tsx` for route segment error handling (creates Error Boundaries).
     -   Use `page.tsx` as the primary UI for a route segment.

 8.  **Optimization:**
     -   Use `next/image` for automatic image optimization. Provide `width` and `height`.
     -   Use `next/font` for optimized web font loading.
     -   Use `next/dynamic` with `ssr: false` for components that only work client-side or to code-split large Client Components.
     -   Use `generateStaticParams` in dynamic route segments (`/app/blog/[slug]/page.tsx`) for SSG (Static Site Generation) where appropriate.

 --- TYPESCRIPT ---

 9.  **Strictness:**
     -   Enable `strict: true` in `tsconfig.json`. Address any resulting errors.

 10. **Typing:**
     -   **Avoid `any`:** Use `unknown` for unknown types and perform type checking/narrowing. Use specific types whenever possible.
     -   **Explicit Types:** Define types for function parameters, return values, variables, component props, and state.
     -   **Interfaces vs. Types:**
         -   Use `interface` for defining object shapes (e.g., component props, API responses). They can be extended.
         -   Use `type` for unions, intersections, primitives, tuples, mapped types, or utility types.
     -   **Component Props:** Define `Props` interfaces/types for every component: `interface MyComponentProps { userId: string; }`. Use `React.FC<Props>` or define props directly: `const MyComponent = ({ userId }: MyComponentProps) => { ... };`.
     -   **Generics:** Use generics `<T>` for reusable functions, types, or components working with various data types.
     -   **Utility Types:** Leverage built-in utility types like `Partial<T>`, `Required<T>`, `Readonly<T>`, `Pick<T, K>`, `Omit<T, K>`.

 11. **Type Assertions:**
     -   Avoid type assertions (`value as Type`) unless absolutely necessary and you are certain of the type. Prefer type guards or narrowing.
     -   Use `satisfies` operator to validate an expression against a type without changing its underlying type.

 --- SUPABASE ---

 12. **Client Initialization:**
     -   **Use `@supabase/ssr`:** Employ the helper functions (`createClientComponentClient`, `createServerComponentClient`, `createRouteHandlerClient`, `createMiddlewareClient` from `@supabase/ssr`) to manage Supabase client instances and auth state correctly across different Next.js rendering environments (Client/Server Components, Middleware, Route Handlers).
     -   **Typed Client:** Initialize the client with database types generated via Supabase CLI for full type safety: `createClient<Database>(...)`.

 13. **Database Types:**
     -   **Generate Types:** Use `supabase gen types typescript --project-id <your-project-ref> --schema public > types/supabase.ts` (or similar command) to generate TS types from your DB schema.
     -   **Import & Use:** Import the generated `Database` type and `Json` type (if using JSON columns) into your Supabase client setup and throughout your application when interacting with Supabase data. `import { Database } from '@/types/supabase';`

 14. **Data Access:**
     -   **Row Level Security (RLS):** **ALWAYS ENABLE RLS** on tables containing sensitive data. Define policies carefully based on `auth.uid()` or user roles. Test RLS policies thoroughly. Do not rely solely on client-side checks.
     -   **Specific Selects:** Avoid `select('*')`. Explicitly list the columns you need: `supabase.from('posts').select('id, title, author:profiles(id, username)')`. This improves performance and type safety.
     -   **Error Handling:** Always check for errors returned from Supabase calls: `const { data, error } = await supabase.from(...).select(...); if (error) { console.error(error); throw new Error(...) }`. Handle errors gracefully.
     -   **Server-Side Logic:** Prefer performing sensitive operations (inserts, updates, deletes with complex logic or using `service_role` key) on the server (Server Components, Server Actions, Route Handlers) rather than directly from the client using only the `anon` key.

 15. **Authentication:**
     -   Use Supabase Auth UI components or implement authentication flows using Supabase JS client methods (`signInWithPassword`, `signUp`, `signOut`, `onAuthStateChange`, etc.).
     -   Manage auth state using `@supabase/ssr` helpers and React Context or a state management library if needed across many Client Components.
     -   Secure routes/pages based on authentication status, preferably using server-side checks or middleware.

 16. **Supabase Edge Functions:**
     -   Use for complex backend logic, integrations, or operations needing proximity to the database, especially when triggered by webhooks or database events. Type function inputs and outputs.

 --- INTEGRATION & PATTERNS ---

 17. **Server/Client Data Flow:** Fetch data in Server Components using the server client. Pass required, typed data down as props to Client Components. Avoid passing the full Supabase client instance to Client Components unless strictly necessary and carefully managed (e.g., for real-time subscriptions).

 18. **Mutations Pattern:**
     -   Client Component triggers a Server Action.
     -   Server Action executes Supabase mutation (insert/update/delete) using the server client.
     -   Server Action handles errors and potentially returns data.
     -   Server Action calls `revalidatePath` or `revalidateTag` to update the Next.js cache.
     -   Client Component updates UI based on Server Action result (e.g., using `useOptimistic`, showing success/error messages).

 19. **Type Safety Chain:** Ensure type safety flows from Supabase (generated types) -> Server Components/Route Handlers/Server Actions -> Client Component props.

 --- CODE QUALITY ---

 20. **Error Handling:** Implement robust error handling on both client and server. Use `error.tsx` in Next.js, `try...catch` blocks, and handle Supabase errors explicitly. Log errors effectively.
 21. **Input Validation:** Validate all user input on the server (Server Actions, Route Handlers) before database interaction (use libraries like Zod).
 22. **Comments:** Write clear comments explaining *why* code exists, especially for complex logic, workarounds, or non-obvious decisions. Copilot uses comments for context.
 23. **Consistency:** Maintain consistent coding style, formatting (use Prettier/ESLint), and patterns throughout the project.
 24. **Testing:** Write tests (unit, integration, e2e) for critical logic, especially involving authentication and data manipulation.
*/
