# React Advanced

## 1. Rendering
    
리액트에서 화면의 렌더링을 야기하는 요인들은 다음과 같다.
- state(setState) 의 변경
- props 의 변화
- 중앙 관리 상태의 변화 (Redux, Recoil...)
- 부모 컴포넌트의 리렌더링

따라서 무분별하게 setState 나 부모자식관계를 설정한다면 불필요한 리렌더링이 일어나 성능저하를 야기할 수 밖에 없기 때문에, 우리는 여러 방법을 통하여 렌더링 최적화를 실시 해야 한다,

렌더링 최적화의 대표적인 방법 중 하나가 바로 `useMemo` 이다.

### useMemo
React Hook 중 하나로, CPU 연산이 많은 함수를 캐싱하기 위해 사용한다.
```typescript
export default function Rendering() {
  const [immutable] = useState<number>(0);
  const [count, setCount] = useState<number>(0);

  const increment = () => {
    console.log('increment');
    setCount(count + 1);
  };
  const calculate = () => {
    console.log('do calculating...');
    return (immutable + 2 * 2 * 1000 * 27) / 28;
  };

  return (
    <section>
      <h2>Count: {count}</h2>
      <button onClick={increment}>Increment</button>
      <p>Calculate value : {calculate()}</p>
    </section>
  );
}
```
위 코드를 실행해 보면, 버튼을 클릭할때마다 calculate 함수가 재실행되는 것을 콘솔을 통해 알 수 있다.

그런데 버튼을 클릭해도 immutable 의 값은 변하지 않기 때문에 calculate 의 반환값 역시 같다.
즉, 렌더링할때마다 불필요한 연산을 계속해서 실시하고 있는 것이다.

이러한 상황에서 `useMemo`를 사용 할 수 있다. 
```typescript
export default function Rendering() {
    const [immutable, setImmutable] = useState<number>(0);
    const [count, setCount] = useState<number>(0);

    const increment = () => {
        console.log('increment');
        setCount(count + 1);
    };
    const changeImmutable = () => {
        console.log('changeImmutable');
        setImmutable(immutable + 1);
    };

    // 불필요한 연산을 useMemo로 최적화
    const calculate = useMemo(() => {
        console.log('do calculating using useMemo...');
        return (immutable + 2 * 2 * 1000 * 27) / 28;
    }, [immutable]);

    return (
        <section>
            <h2>Count: {count}</h2>
    <div style={{ display: 'flex', gap: '20px' }}>
    <button onClick={increment}>Increment</button>
        <button onClick={changeImmutable}>Change Immutable</button>
    </div>

    <p>Calculate value : {calculate}</p>
    </section>
);
}
```
이렇게 `useMemo`를 사용하여 결과값을 캐싱해주어, dependency array 에 들어있는 변수의 값이 변하지 않는다면, 해당 함수는 동일한 결과값을 가지는 연산을 다시 실시 하지 않는다.

실제로 increment 버튼을 클릭시 `calculate`가 다시 연산되지 않는데 반하여,  change Immutable 버튼을 클릭시, dependency array 의 값이 바뀌기 때문에 연산을 다시 실시하는것을 볼 수 있다.  