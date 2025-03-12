### 이미지 최적화

https://nextjs.org/docs/app/api-reference/components/image

```jsx
import Image from "next/image";
<Image src={logoImg} alt="logo" priority />;
```

✅  next/image의 장점

- fill 속성을 사용하면 부모 컨테이너에 맞춰 유연한 이미지 크기를 적용
- 기본적으로 지연 로딩(lazy loading) 이 활성화되어 있어, 사용자가 스크롤할 때만 이미지를 로드하여 불필요한 데이터 로드를 줄임
- 사용자의 브라우저에서 지원하는 최적의 이미지 포맷(WebP, AVIF 등)으로 변환하여 전송하여 성능을 개선

### Suspense을 활용한 데이터 로딩

```jsx
...
import { Suspense } from "react";

export default function MealsPage() {
  return (
    <>
		 ...
        <Suspense
          fallback={<p className={classes.loading}>Fetching meals...</p>}
        >
          <Meals />
        </Suspense>
		...
    </>
  );
}
```

✅  Suspense의 장점

- 서버에서 데이터를 불러오는 동안 fallback UI를 적용하여 사용자 경험 개선
- 클라이언트에서 useEffect 없이 자연스럽게 데이터 로딩 처리 가능

### error.js를 활용한 Error handling

Next.js에서는 error.js 파일을 통해 클라이언트 컴포넌트에서 발생한 오류 처리 가능 (이때, error.js는 클라이언트 컴포넌트여야 하는데, 이는 error.js가 React Error Boundary를 사용하기 때문인데, 즉, 페이지가 렌더링 된 후 클라이언트 측에서 오류를 잡아내기 때문이다.)

```jsx
"use client";

export default function Error({ error }) {
  return (
    <main className="error">
      <h1>An error occured!</h1>
      <p>Failed to fetch meal data. Please try again later.</p>
    </main>
  );
}
```

✅  Next.js의 에러 처리 방법

- error.js: 특정 페이지의 오류 처리
- non-found.js: 404 페이지 처리
- notFound() 함수: 특정 조건에서 404 페이지로 리다이렉션

  ```
  import { notFound } from "next/navigation";

  export default function MealDetailPage({ params }) {
    const meal = getMeal(params.mealSlug);

    if (!meal) {
      notFound(); // 가까운 not-found나 오류화면을 보여준다.
    }
  ```

### Server Action을 활용한 데이터 처리

Next.js의 server action을 사용하면 별도의 API 라우트 없이 Next.js 서버에서 데이터를 직접 처리할 수 있다.

```jsx
"use server";
...
export async function shareMeal(prevState, formData) {
  const meal = {
    title: formData.get("title"),
    summary: formData.get("summary"),
    instructions: formData.get("instructions"),
    image: formData.get("image"),
    creator: formData.get("name"),
    creator_email: formData.get("email"),
  };
	...
  await saveMeal(meal);
	...
}
```

✅  Server action의 장점

- 별도의 서버 구축이 필요 없음
- 클라이언트에서 fetch 없이 Next.js 서버에서 직접 데이터 처리 가능
- 보안 강화를 위해 API 키나 DB 연결 정보를 숨길 수 있음

(하지만 분리된 백엔드가 있다면, server action을 사용하지 않고, 프론트엔드에서 fetch 가능)

### useFormStatus()를 활용한 상태 관리

```jsx
"use client";

import { useFormStatus } from "react-dom";

export default function MealsFormSubmit() {
  const { pending } = useFormStatus();
  return (
    <button disabled={pending}>
      {pending ? "Submitting..." : "Share Meal"}
    </button>
  );
}
```

✅  useFormStatus()의 장점

- form이 제출 중인지 쉽게 확인 가능
- 버튼 비활성화 및 상태 표시 쉽게 적용 가능

### revalidatePath()를 활용한 데이터 갱신

Next.js는 aggressive caching 전략을 사용하여 정적인 페이지를 미리 생성한다. Next.js는 정적인 페이지(SSG)를 생성하면 해당 페이지를 캐시하여 빠르게 제공한다. 이로 인해 데이터가 업데이트 되지 않는 문제가 발생할 수 있다.

예를 들어, 명령어 npm run build → npm start를 실행한 프로덕션 환경에서 새로운 데이터인 meal을 생성했다. 이후에 다시 ‘/meals’ 페이지의 meals 목록에 새롭게 추가한 meal이 반영되어 목록이 업데이트 되었을 것이라고 예상했는데, Next.js는 build하면서 사전에 만들 수 있는 페이지를 모두 만들고 caching하기 때문에 meals 목록을 업데이트 하는 API를 호출하지 않았다. 그래서 revalidatePath()를 사용하여 특정 경로의 캐시를 무효화했다.

```jsx
await saveMeal(meal);
revalidatePath("/meals");
redirect("/meals");
```
