# Tori Wallet - Copilot Instructions

이 문서는 GitHub Copilot이 Tori Wallet 프로젝트에서 효과적으로 코드를 생성하도록 돕는 지침입니다.

## 프로젝트 개요

Tori Wallet은 React Native + TypeScript 기반의 Web3 모바일 지갑 앱입니다.

## 기술 스택

- **Framework**: React Native 0.83 (TypeScript)
- **Blockchain**: Viem (이더리움 인터랙션)
- **dApp 연결**: WalletConnect Web3Wallet
- **상태 관리**: Zustand
- **서버 상태**: TanStack Query (React Query)
- **스타일링**: styled-components/native
- **네비게이션**: React Navigation 7
- **테스트**: Jest + React Testing Library + Detox

## 코딩 컨벤션

### 파일 구조

- 컴포넌트: `PascalCase.tsx`
- 훅: `useCamelCase.ts`
- 서비스: `camelCase.ts`
- 유틸: `camelCase.ts`
- 타입: 같은 파일 또는 `types.ts`

### 컴포넌트 작성

```tsx
// 함수형 컴포넌트 + styled-components 사용
import React from 'react';
import styled from 'styled-components/native';

interface Props {
  title: string;
  onPress?: () => void;
}

export function MyComponent({ title, onPress }: Props) {
  return (
    <Container onPress={onPress}>
      <Title>{title}</Title>
    </Container>
  );
}

const Container = styled.TouchableOpacity`
  padding: ${({ theme }) => theme.spacing.md}px;
`;

const Title = styled.Text`
  color: ${({ theme }) => theme.colors.textPrimary};
`;
```

### 훅 작성

```tsx
import { useCallback } from 'react';
import { useQuery, useMutation } from '@tanstack/react-query';

export function useCustomHook() {
  // 상태 조회는 useQuery
  const { data, isLoading } = useQuery({
    queryKey: ['key'],
    queryFn: async () => {},
  });

  // 상태 변경은 useMutation
  const mutation = useMutation({
    mutationFn: async () => {},
  });

  const action = useCallback(() => {
    // ...
  }, []);

  return { data, isLoading, action };
}
```

### 서비스 작성

```tsx
// 싱글톤 클래스 패턴
class MyService {
  private cache: Map<string, unknown> = new Map();

  async getData(id: string): Promise<Data> {
    // ...
  }
}

export const myService = new MyService();
```

### 에러 처리

```tsx
import { createAppError, ErrorCode, toAppError, logError } from '@utils/error';

try {
  await riskyOperation();
} catch (error) {
  logError(error, 'context');
  throw toAppError(error);
}
```

## 테마 사용

```tsx
// theme.ts의 값들 사용
${({theme}) => theme.colors.primary}
${({theme}) => theme.spacing.md}px
${({theme}) => theme.borderRadius.lg}px
```

### 사용 가능한 색상

- `primary`, `primaryLight`, `primaryDark`
- `secondary`, `secondaryLight`, `secondaryDark`
- `background`, `backgroundSecondary`, `backgroundTertiary`, `surface`
- `textPrimary`, `textSecondary`, `textTertiary`, `textMuted`
- `success`, `warning`, `error`, `info`
- `border`, `borderLight`

### 간격

- `xs: 4`, `sm: 8`, `md: 16`, `lg: 24`, `xl: 32`, `xxl: 48`

### Border Radius

- `sm: 8`, `md: 12`, `lg: 16`, `xl: 24`, `full: 9999`

## 지갑 관련 패턴

### 주소 형식

```tsx
// viem의 타입 사용
const address: `0x${string}` = '0x1234...';
```

### 금액 처리

```tsx
import { formatEther, parseEther } from 'viem';

// Wei -> ETH
const eth = formatEther(weiAmount);

// ETH -> Wei
const wei = parseEther('1.5');
```

### 체인 ID

```tsx
// 지원하는 체인
const SUPPORTED_CHAINS = {
  ethereum: 1,
  sepolia: 11155111,
  polygon: 137,
  arbitrum: 42161,
  optimism: 10,
  base: 8453,
};
```

## Import 순서

1. React, React Native
2. 외부 라이브러리
3. 내부 모듈 (@로 시작하는 별칭)
4. 상대 경로 임포트
5. 타입 임포트

## 보안 주의사항

- 니모닉/개인키는 절대 콘솔에 로깅하지 않음
- 민감한 데이터는 `react-native-keychain` 사용
- API 키는 환경변수로 관리

## 테스트 작성

```tsx
import { render, screen, fireEvent } from '@testing-library/react-native';

describe('Component', () => {
  it('should render correctly', () => {
    render(<Component />);
    expect(screen.getByText('Title')).toBeTruthy();
  });
});
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kwakhyun) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
