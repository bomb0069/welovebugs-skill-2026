# TypeScript / JavaScript Test Guide

Code patterns for generating tests with Jest or Vitest.

## Dependencies

```json
// package.json (Jest)
"devDependencies": {
  "jest": "^29.0.0",
  "@types/jest": "^29.0.0",
  "ts-jest": "^29.0.0"
}

// package.json (Vitest)
"devDependencies": {
  "vitest": "^1.0.0"
}
```

## Auto-detect: Jest vs Vitest

- `jest.config.*` exists → use Jest
- `vitest.config.*` exists or `vitest` in package.json → use Vitest
- Both → ask user
- Neither → default to Jest

## Test file structure

### Jest

```typescript
import { UserRegistrationService } from './UserRegistrationService';
import { UserRepository } from './UserRepository';

jest.mock('./UserRepository');

describe('UserRegistrationService', () => {
  let service: UserRegistrationService;
  let mockRepository: jest.Mocked<UserRepository>;

  beforeEach(() => {
    mockRepository = new UserRepository() as jest.Mocked<UserRepository>;
    service = new UserRegistrationService(mockRepository);
  });

  // Test groups go here
});
```

### Vitest

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { UserRegistrationService } from './UserRegistrationService';
import { UserRepository } from './UserRepository';

vi.mock('./UserRepository');

describe('UserRegistrationService', () => {
  let service: UserRegistrationService;
  let mockRepository: UserRepository;

  beforeEach(() => {
    mockRepository = new UserRepository();
    vi.mocked(mockRepository);
    service = new UserRegistrationService(mockRepository);
  });

  // Test groups go here
});
```

## Traceability in test reports

Use the test name and describe blocks to show traceability in the report.
Add a traceability comment block at the top of each test:

```typescript
// Traceability constants
const TRACEABILITY = {
  requirement: 'User must be aged 18-65 to register',
  source: 'test-cases/user-registration-age.md',
  level: 'Unit Test',
};

describe('UserRegistrationService', () => {
  describe('Age Validation (BVA) | Requirement: User must be aged 18-65', () => {

    it('UT-01: should accept registration when age is at minimum boundary (18) [BVA:Min]', () => {
      // Requirement: User must be aged 18-65 to register
      // TC-ID: UT-01 | Technique: BVA (C1:Min)
      // Source: test-cases/user-registration-age.md | Level: Unit Test
      ...
    });
  });
});
```

This produces a report like:
```
✓ UserRegistrationService
  ✓ Age Validation (BVA) | Requirement: User must be aged 18-65
    ✓ UT-01: should accept registration when age is at minimum boundary (18) [BVA:Min]
    ✗ UT-06: should reject registration when age is above maximum (66) [BVA:AboveMax]
```

## Test method patterns

### Single test

```typescript
// Jest
it('UT-01: should accept registration when age is at minimum boundary (18) [BVA:Min]', () => {
  // Requirement: User must be aged 18-65 to register
  // TC-ID: UT-01 | Technique: BVA (C1:Min)
  const request = { name: 'Sarah Lee', age: 18, email: 'sarah@test.com' };

  // When
  const result = service.register(request);

  // Then
  expect(result.accepted).toBe(true);
  expect(result.message).toBe('Registration accepted, welcome page shown');
});
```

### Parameterized test (test.each)

```typescript
it.each([
  { utId: 'UT-02', age: 18, description: 'Min boundary' },
  { utId: 'UT-03', age: 19, description: 'Above min' },
  { utId: 'UT-04', age: 64, description: 'Below max' },
  { utId: 'UT-05', age: 65, description: 'Max boundary' },
])('$utId: should accept when age is $age ($description)', ({ age }) => {
  const request = { name: 'Test User', age, email: 'test@test.com' };
  const result = service.register(request);
  expect(result.accepted).toBe(true);
});
```

### Error/exception test

```typescript
it('should throw ValidationError when age is below minimum (17)', () => {
  // Given - UT-01: Age below minimum (17)
  const request = { name: 'Tom Park', age: 17, email: 'tom@test.com' };

  // When & Then
  expect(() => service.register(request))
    .toThrow('Age must be between 18 and 65');
});

// Async
it('should reject when age is invalid', async () => {
  const request = { name: 'Tom Park', age: 17, email: 'tom@test.com' };
  await expect(service.register(request)).rejects.toThrow('Age must be between 18 and 65');
});
```

## Grouping with describe

```typescript
describe('UserRegistrationService', () => {

  describe('Age Validation (BVA)', () => {
    describe('valid boundaries', () => {
      it.each([18, 19, 64, 65])('should accept when age is %i', (age) => { ... });
    });

    describe('invalid boundaries', () => {
      it.each([17, 66])('should reject when age is %i', (age) => { ... });
    });
  });

  describe('File Type Validation (EP)', () => {
    describe('valid partitions', () => { ... });
    describe('invalid partitions', () => { ... });
  });

  describe('State Transitions', () => {
    describe('valid transitions', () => { ... });
    describe('invalid transitions', () => { ... });
  });
});
```

## Mocking patterns

### Jest

```typescript
// Mock module
jest.mock('./PaymentGateway');

// Mock return value
mockRepository.findByEmail.mockResolvedValue(null);
mockRepository.save.mockResolvedValue(savedUser);

// Verify
expect(mockRepository.save).toHaveBeenCalledWith(
  expect.objectContaining({ name: 'Sarah Lee', age: 18 })
);
expect(mockRepository.save).toHaveBeenCalledTimes(1);
```

### Vitest

```typescript
vi.mock('./PaymentGateway');

vi.mocked(mockRepository.findByEmail).mockResolvedValue(null);

expect(mockRepository.save).toHaveBeenCalledWith(
  expect.objectContaining({ name: 'Sarah Lee', age: 18 })
);
```

## Assertion patterns

```typescript
// Boolean
expect(result.accepted).toBe(true);
expect(result.accepted).toBeFalsy();

// String
expect(result.message).toBe('Expected message');
expect(result.message).toContain('partial match');

// Number
expect(result.amount).toBe(30000);
expect(result.age).toBeGreaterThanOrEqual(18);

// Object
expect(result).toEqual({ accepted: true, message: 'OK' });
expect(result).toMatchObject({ accepted: true });

// Array
expect(result.errors).toHaveLength(1);
expect(result.errors).toContain('Age must be between 18 and 65');

// Exception
expect(() => fn()).toThrow('message');
expect(() => fn()).toThrow(ValidationError);
```

## Test file location

- Colocated: `src/services/UserRegistrationService.test.ts`
- Separate: `__tests__/services/UserRegistrationService.test.ts`
- Check project convention and follow it

## Naming convention

Test names use plain English: `'should accept registration when age is at minimum boundary'`
