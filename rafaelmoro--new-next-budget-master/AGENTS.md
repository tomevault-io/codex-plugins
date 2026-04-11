For the unit tests, consider the following instructions:

- If the component has a hook useDashboardStore, it means it has a zustand store, then take as reference the file \_\_tests\_\_/feature/Dashboard/Dashboard.test.tsx and wrap the component with DashboardStoreProvider and use mockAccounts

- If you get the error "useDashboardStore must be used within DashboardStoreContext", it means it has a zustand store, then take as reference the file \_\_tests\_\_/feature/Dashboard/Dashboard.test.tsx and wrap the component with DashboardStoreProvider and use mockAccounts

- If in the terminal has an error related to the cookies, take as reference \_\_tests\_\_/home.test.tsx to mock the cookies like shown in lines 13 to 18 using jest.mock of next/headers

- If the tests fails because of the router like this error "invariant expected app router to be mounted" or if the component uses router from next, take as reference \_\_tests\_\_/home.test.tsx to wrap the component in AppRouterContextProviderMock like shown there. Add the function push as jest.fn()

- If the component to be testes has any imports related to tanstack query like useQuery or useMutation, use the QueryProviderWrapper located in src/app/QueryProviderWrapper.tsx

- If the test fail like this error "No QueryClient set, use QueryClientProvider to set one" because it needs a query provider due tanstack query usage of mutation or query, takes as reference \_\_tests\_\_/home.test.tsx to wrap the component in QueryProviderMock like shown there

- If the component to be tested uses router from next/navigation, then wrap the component into AppRouterContextProviderMock located in src/shared/ui/organisms/AppRouterContextProviderMock.tsx

- If the test has an error related to "ResizeObserver is not defined", then use the mock of the file \_\_tests\_\_/feature/Dashboard/Overview/StatisticsSubscreen.test.tsx

- Do not Mock the component to avoid rendering its internals

- Remember to use userEvent for user interactions. Do not use fireEvent.

- Do not write tests asserting CSS classes or style classes in components. Focus on testing behavior, functionality, and rendered content instead.

- Do not extract container from render (e.g., `const { container } = render(...)`). Do not use querySelector or other DOM query methods. Always use `screen` from React Testing Library to query elements.

- Do not mock custom hooks like shown in the snippet below

- Do not mock the sonner library (e.g., Toaster, toast) in your tests.
- Do not mock any utility functions or modules (e.g., do not mock "@/shared/utils/records.utils" or similar).
- Do not mock next/navigation (e.g., useRouter) in your tests.
- Do not use require() imports inside your tests; always use static import statements at the top of the file.

```typescript
// Mock the hook
jest.mock("@/shared/hooks/useTransferBankAccounts");
```

- Make sure the tests are passing. Iterate until the tests are passing. Apply the changes into the file if needed. To be sure these are passing run the command: pnpm test -- tests/feature/Accounts/CreateAccButton.test.tsx

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RafaelMoro)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/RafaelMoro)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
