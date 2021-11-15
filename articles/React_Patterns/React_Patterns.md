# Хорошие практики при проектировании React-компонентов
## Нерасширяемый пример
```jsx
function MySelect({onClick}) {
    const [data, setData] = useState([]);
    const [error, setError] = useState(false);
    const [fetching, setFetching] = useState(false);
    const [value, setValue] = useState("");

    const handleChange = (value) => {
        setValue(value);
        onClick(value);
    };

    useEffect(() => {
        setFetching(true);
        getData()
            .then((res) => {
                setFetching(false);
                setData(res);
            })
            .catch((err) => {
                setFetching(false);
                setError(true);
            });
    }, []);

    if (fetching) {
        return <div>Please, wait...</div>;
    }

    if (error) {
        return <div>Error:(</div>;
    }

    return (
        <select value={value} onChange={(e) => handleChange(e.target.value)}>
            {data.map((item) => {
                return <option value={item.id}>{item.text}</option>;
            })}
        </select>
    );
}
```

Стили

```css
.MySelect {
    margin: 10px;
    width: 200px;
    border: 4px dashed royalblue;
    color: darkblue
}
```

## Разделение данных и представления

```jsx
export default function MySelectContainer({onChange}) {
    // ...
    return <MySelect items={data} value={value} onChange={handleChange}/>;
}

function MySelect({items, value, onChange}) {
    return (
        <select
            className="MySelect"
            value={value}
            onChange={(e) => onChange(e.target.value)}
        >
            {items.map((item) => {
                return <option value={item.id}>{item.text}</option>;
            })}
        </select>
    );
}
```

## Открытость компонента для расширения

```js
function MySelect({items, ...props}) {
    return (
        <select
            {...props}
            className={clsx("MySelect", props.className)}
        >
            {items.map((item) => {
                return <option key={item.id} value={item.id}>{item.text}</option>;
            })}
        </select>
    );
}
```

## Позиционирование компонента снаружи

```css
.MySelect {
    width: 100%;
    border: 4px dashed royalblue;
    color: darkblue
}
```

## Гибкость использования `children`

```js
function MySelect({children, ...props}) {
    return (
        <select {...props} className={clsx("MySelect", props.className)}>
            {children}
        </select>
    );
}

<MySelect value={value} onChange={handleChange}>
    {data.map((item) => {
        return <option key={item.id} value={item.id}>{item.text}</option>;
    })}
</MySelect>
```

## Выделение логики в кастомный хук

![state](pics/state.png)

```jsx
const useMySelect = () => {
    const [prefix, setPrefix] = useState("");
    return {
        prefix,
        setPrefix
    };
};
```


```jsx
function Option({children, prefix, ...props}) {
    return (
        <option {...props} className={clsx("Option", props.className)}>
            {prefix}
            {children}
        </option>
    );
}

function MySelect({children, onPrefixChange, prefixValue, ...props}) {
    return (
        <>
            <label>
                prefix:
                <input type="text" onChange={onPrefixChange} value={prefixValue}/>
            </label>
            <select {...props} className={clsx("MySelect", props.className)}>
                {children}
            </select>
        </>
    );
}

function MySelectContainer() {
    const {prefix, setPrefix} = useMySelect();

    return (
        <MySelect
            className="MySelect"
            value={value}
            onChange={handleChange}
            prefixValue={prefix}
            onPrefixChange={(e) => setPrefix(e.target.value)}
        >
            {data.map((item) => {
                return (
                    <Option key={item.id} value={item.id} prefix={prefix}>
                        {item.text}
                    </Option>
                );
            })}
        </MySelect>
    );
}
```

### `Prop getter`

```jsx
const useMySelect = () => {
    const [prefix, setPrefix] = useState("");

    const getSelectProps = (props) => ({
        ...props,
        onPrefixChange: (e) => {
            if (props.onPrefixChange) {
                props.onPrefixChange(e);
            }
            setPrefix(e.target.value);
        },
        prefixValue: props.prefixValue ? props.prefixValue : prefix
    });

    const getOptionProps = (props) => ({
        ...props,
        prefix: props.prefixValue ? props.prefixValue : prefix
    });

    return {
        getSelectProps,
        getOptionProps
    };
};
```
```jsx

const {getOptionProps, getSelectProps} = useMySelect();
return (
    <MySelect
        {...getSelectProps({
            value,
            onChange: handleChange,
        })}
    >
        {data.map((item) => {
            return (
                <Option key={item.id} {...getOptionProps({value: item.id})}>
                    {item.text}
                </Option>
            );
        })}
    </MySelect>
)
```

## Получение данных

```jsx
    const [data, setData] = useState([]);
    const [error, setError] = useState(false);
    const [fetching, setFetching] = useState(false);

    useEffect(() => {
        setFetching(true);
        getData()
            .then((res) => {
                setFetching(false);
                setData(res);
            })
            .catch((err) => {
                setFetching(false);
                setError(true);
            });
    }, []);
```

```jsx
  const [data, setData] = useState([]);
  const [state, setState] = useState('initial');
  
  const { getOptionProps, getSelectProps } = useMySelect();

  useEffect(() => {
    setState('fetching');
    getData()
      .then((res) => {
        setState('success');
        setData(res);
      })
      .catch((err) => {
        setState('error')
      });
  }, []);

  if (state === 'fetching') {
    return <div>Please, wait...</div>;
  }

  if (state === 'error') {
    return <div>Error:(</div>;
  }
```









