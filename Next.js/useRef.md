# 예시
## yoon-pf에서
공식 문서에서는 `app.js`에서 useRef를 사용해 quillRef를 만들어 `editor.js`로 전달하던데,
```jsx
const Delta = Quill.import('delta');

const App = () => {
  const [range, setRange] = useState();
  const [lastChange, setLastChange] = useState();
  const [readOnly, setReadOnly] = useState(false);

  // Use a ref to access the quill instance directly
  const quillRef = useRef();

  return (
    <div>
      <Editor
        ref={quillRef}
        readOnly={readOnly}
        defaultValue={new Delta()
          .insert('Hello')
          .insert('\n', { header: 1 })
          .insert('Some ')
          .insert('initial', { bold: true })
          .insert(' ')
          .insert('content', { underline: true })
          .insert('\n')}
        onSelectionChange={setRange}
        onTextChange={setLastChange}
      />
      <div class="controls">
        <label>
          Read Only:{' '}
          <input
            type="checkbox"
            value={readOnly}
            onChange={(e) => setReadOnly(e.target.checked)}
          />
        </label>
        <button
          className="controls-right"
          type="button"
          onClick={() => {
            alert(quillRef.current?.getLength());
          }}
        >
          Get Content Length
        </button>
      </div>
      <div className="state">
        <div className="state-title">Current Range:</div>
        {range ? JSON.stringify(range) : 'Empty'}
      </div>
      <div className="state">
        <div className="state-title">Last Change:</div>
        {lastChange ? JSON.stringify(lastChange.ops) : 'Empty'}
      </div>
    </div>
  );
};

export default App;
```
근데 클로드가 typescript용으로 짜달라고 하니까 페이지에서 useRef를 하지 않고 만들었음.
그냥 페이지에서 `<Editor onTextChange={handleTextChange} />` 만 써서 호출함.

ref 사용
장점:
1. **직접적인 DOM/인스턴스 접근**: 부모 컴포넌트에서 Quill 인스턴스나 DOM 요소에 직접 접근할 수 있습니다.
2. **더 많은 제어권**: 부모 컴포넌트에서 에디터의 기능을 직접 제어할 수 있습니다.
3. **성능**: 상태 업데이트 없이 DOM을 직접 조작할 수 있어 리렌더링이 줄어듭니다.
4. **라이브러리 패턴 호환성**: 많은 JavaScript 라이브러리들이 이 패턴을 기대합니다.
 단점:
5. **복잡성 증가**: ref 관리와 forwardRef 사용으로 코드가 복잡해집니다.
6. **React 철학과의 충돌**: React의 선언적 패러다임과 다소 어긋납니다.
7. **테스트 어려움**: DOM에 직접 접근하는 코드는 테스트하기 어려울 수 있습니다.

 ref 비사용
장점:
1. **단순성**: 코드가 더 간단하고 이해하기 쉽습니다.
2. **캡슐화**: 컴포넌트가 자체 상태를 관리하므로 더 독립적입니다.
3. **React 방식**: 상태와 props로 통신하는 React의 철학에 맞습니다.
4. **테스트 용이성**: 상태와 props 기반이라 테스트하기 쉽습니다.
단점:
5. **제한된 제어**: 부모에서 에디터의 세부 기능에 접근하기 어렵습니다.
6. **성능 오버헤드**: 상태 업데이트로 인한 리렌더링이 발생할 수 있습니다.

ref는 