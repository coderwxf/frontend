通过`useImperativeHandle`自定义向外暴露的属性和方法

```jsx
import { useEffect, useRef, forwardRef, useImperativeHandle } from "react"

const Child = forwardRef((props, ref) => {
  // useImperativeHandle(ref对象，callback)
  // callback的返回值 会被赋值给ref.current
  useImperativeHandle(ref, () => {
    return {
      tag: 'child',
      focus: () => {
        console.log('focus')
      }
    }
  })

  return <div> Child </div>
})

export default function Demo() {
  const box = useRef(null)

  useEffect(() => {
    console.log(box.current.tag)
    box.current.focus()
  }, [])

  return (
    <div>
      <Child ref={box} />
    </div>
  )
}
```

