# FORM_RULES — 폼 구현 패턴

## 스택

- **react-hook-form** `^7.53.0` — 폼 상태 관리
- **@hookform/resolvers** `^3.9.0` — Zod 스키마 연동
- **zod** `^3.23.8` — 유효성 스키마 정의

---

## Zod 스키마

파일: `src/utils/validations.ts`

새 스키마는 이 파일에 추가한다.

```ts
// 현재 정의된 스키마
export const loginSchema = z.object({
  accountName: z.string().min(1, '아이디를 입력해주세요.'),
  password: z.string().min(6, '비밀번호는 6자 이상이어야 합니다.'),
});

export const userSchema = z.object({
  name: z.string().min(1, '이름을 입력해주세요.'),
  email: z.string().email('유효한 이메일을 입력해주세요.'),
  role: z.string().min(1, '역할을 선택해주세요.'),
  isActive: z.string(),
});
```

---

## useForm 초기화

```ts
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { userSchema } from '@/utils/validations';

const form = useForm<z.infer<typeof userSchema>>({
  resolver: zodResolver(userSchema),
  defaultValues: {
    name: '',
    email: '',
    role: '',
    isActive: 'true',
  },
});
```

---

## 수정(Edit) 모드 — reset 패턴

API로 데이터를 불러온 뒤 `form.reset()`으로 초기값 세팅.

```ts
const { data } = useGetUserDetail(id ?? '');

useEffect(() => {
  if (data) {
    form.reset({
      name: data.name,
      email: data.email,
      role: data.role,
      isActive: String(data.isActive),
    });
  }
}, [data, form]);
```

- `isActive`가 `boolean`으로 오는 경우 `String()` 변환 필요 (Select value는 string).
- `enabled: !!id` 옵션: id가 없으면 API 호출 안 함.

---

## FormFields — RHF 연동 컴포넌트

`Form` 컴포넌트 안에서만 사용. `form.control`을 반드시 전달.

```tsx
import { Form } from '@/components/ui/form';
import { FormFields, FormFieldType } from '@/components/shared/form-fields';

<Form {...form}>
  <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-5">

    {/* 텍스트 인풋 */}
    <FormFields
      fieldType={FormFieldType.INPUT}
      control={form.control}
      name="name"
      label="이름"
      placeholder="이름을 입력하세요"
    />

    {/* 비밀번호 */}
    <FormFields
      fieldType={FormFieldType.PASSWORD}
      control={form.control}
      name="password"
      label="비밀번호"
      placeholder="비밀번호를 입력하세요"
    />

    {/* 셀렉트 */}
    <FormFields
      fieldType={FormFieldType.SELECT}
      control={form.control}
      name="role"
      label="역할"
      placeholder="역할을 선택하세요"
      options={[
        { label: '관리자', value: 'ADMIN' },
        { label: '일반', value: 'USER' },
      ]}
    />

    <Button type="submit" disabled={form.formState.isSubmitting}>
      저장
    </Button>
  </form>
</Form>
```

**FormFieldType enum:**

| 값 | 렌더 컴포넌트 |
|---|---|
| `INPUT` | `<Input>` |
| `PASSWORD` | `<Input type="password">` |
| `NUMBER` | `<Input type="number">` |
| `TEXTAREA` | `<Textarea>` |
| `SELECT` | `<Select>` + `<SelectContent>` |

---

## FormInput — 독립형 (RHF 없을 때)

RHF 없이 독립적으로 쓰는 인풋. 레이블/에러/헬퍼/카운터를 통합.

```tsx
import { FormInput } from '@/components/shared/FormInput';
// 또는
import { FormInput } from '@/components/shared/form-fields';

const [v, setV] = useState('');

<FormInput
  label="항목명"
  required
  value={v}
  onChange={(e) => setV(e.target.value)}
  maxLength={20}
  showCount
  errorMessage="올바른 값을 입력해 주세요."
  helperText="영문·숫자 조합 8자 이상"
  wrapperClassName="w-[384px]"
/>
```

---

## onSubmit 패턴

```ts
const { mutateAsync: postUser, isPending } = usePostUser();

const onSubmit = async (values: z.infer<typeof userSchema>) => {
  if (isEdit) {
    await updateUser({ uuid: id, ...values });
  } else {
    await postUser(values);
  }
  router.push('/users');
};
```

- Mutation의 `onSuccess`에서 alert + invalidate 처리 → onSubmit에서 중복 처리 불필요.
- `isPending` 상태로 Submit 버튼 `disabled` 처리.

---

## 폼 에러 직접 세팅

API 에러를 특정 필드에 연결할 때:

```ts
form.setError('password', { message: '아이디 또는 비밀번호를 확인해주세요.' });
```

`SignInView`에서 로그인 실패 시 사용하는 패턴.
