# what is diffrence of slug and ...slug?

```tsx
[slug]
[...slug]
```

이 둘의 차이는 무엇일까요?

쉽게 생각하면 ...으로 표기한 경우 모든 segments를 catch한다고 생각할 수 있습니다.

예컨대 /pages/shop/[...slug].js의 경우에는

/shop/a/b/c 와 같이 중첩된 라우팅에서 모든 파라미터를 배열형태로 받아오게됩니다.

[a,b,c]와 같이요!

반면 ...이 없는 경우에는 {slug:'a'}와 같은 형태로 받게됩니다.