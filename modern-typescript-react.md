# Windsurf Code Quality Rules: Modern JavaScript/TypeScript & React

## 🚫 ANTI-PATTERNS TO FLAG (Mark as Code Quality Issues)

### JavaScript/TypeScript Async Patterns

**❌ REJECT these patterns:**
```typescript
// Manual Promise wrapping in async functions
return Promise.resolve(value);
return Promise.reject(new Error('message'));

// Promise chains instead of async/await
someFunction()
  .then((result) => doSomething(result))
  .catch((error) => handleError(error));

// Constructor with unhandled async operations
constructor() {
  this.service.create('data'); // Fire-and-forget async call
}
```

**✅ REQUIRE these patterns:**
```typescript
// Direct returns/throws in async functions
async function example() {
  return value;           // Not Promise.resolve(value)
  throw new Error('msg'); // Not Promise.reject(new Error('msg'))
}

// Async/await instead of chains
try {
  const result = await someFunction();
  const processed = await doSomething(result);
} catch (error) {
  handleError(error);
}

// Async factory pattern for initialization
static async create() {
  const instance = new MyClass();
  await instance.initialize();
  return instance;
}
```

### React Component Patterns

**❌ REJECT these patterns:**
```typescript
// Class components (unless legacy requirement)
class MyComponent extends Component {
  constructor(props) {
    super(props);
    this.handleClick = this.handleClick.bind(this); // Manual binding
  }
  
  componentDidMount() { } // Lifecycle methods
  
  render() { return <div />; }
}

// Promise chains in React components
componentDidMount() {
  api.getData()
    .then(data => this.setState({data}))
    .catch(error => console.error(error));
}
```

**✅ REQUIRE these patterns:**
```typescript
// Functional components with hooks
function MyComponent() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    const fetchData = async () => {
      try {
        const result = await api.getData();
        setData(result);
      } catch (error) {
        console.error(error);
      }
    };
    fetchData();
  }, []);
  
  return <div />;
}
```

### Specific Method Patterns

**❌ FLAG these specific anti-patterns:**
```typescript
// In any method returning Promise<T>:
get(id: string): Promise<T> {
  return Promise.resolve(this.data[id]); // FLAG: Use async/direct return
}

// Any .then()/.catch() chain:
service.method().then(result => { }); // FLAG: Use async/await

// Class component with lifecycle methods:
componentDidMount() { } // FLAG: Use useEffect in functional component
```

## 🔍 DETECTION RULES

1. **Flag any file containing:**
   - `Promise.resolve()` or `Promise.reject()` in async context
   - `.then()` or `.catch()` chains (except in non-async contexts)
   - `class X extends Component` in React files
   - `componentDidMount`, `componentDidUpdate`, etc.
   - Constructor with async calls that aren't awaited

2. **Suggest modern alternatives for:**
   - Class components → Functional components with hooks
   - `componentDidMount` → `useEffect(() => {}, [])`
   - `this.setState` → `useState` hook
   - Promise chains → async/await

## 📝 QUALITY METRICS

**High Quality Modern Code:**
- Uses async/await consistently
- Functional components with hooks
- Direct returns in async functions
- Proper error handling with try/catch
- No manual Promise wrapping

**Low Quality Legacy Code:**
- Manual Promise.resolve/reject calls
- Promise chains (.then/.catch)
- Class components (without specific need)
- Lifecycle methods instead of hooks
- Constructor async operations

## 🎯 MODERNIZATION PRIORITY

1. **Critical:** Class components → Functional components
2. **High:** Promise chains → async/await
3. **Medium:** Manual Promise wrapping → Direct returns
4. **Low:** Constructor patterns → Factory patterns
