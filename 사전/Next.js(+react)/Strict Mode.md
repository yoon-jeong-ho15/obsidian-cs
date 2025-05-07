다음과 같은 개발 전용 동작을 활성화 한다.

- 컴포넌트가 순수하지 않은 렌더링으로 인한 버그를 찾기 위해 추가로 다시 렌더링합니다.
- 컴포넌트가 Effect 클린업이 누락되어 발생한 버그를 찾기 위해 Effect를 다시 실행합니다.
	[[yoon-pf#뷰어에서 에디터가 두개가 됨.]]
- Your components will re-run refs callbacks an extra time to find bugs caused by missing ref cleanup.
- 컴포넌트가 더 이상 사용되지 않는 API를 사용하는지 확인합니다.
