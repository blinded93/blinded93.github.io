---
layout: post
title: "React Class vs. Function Components"
date:       2019-07-13 22:31:45 -0400
permalink: class-vs-function-components
---

In this post, I’m going to explain the differences between class and function components. One of the most prominent patterns I was taught for deciding between class and function components was the container/presentation pattern. In this, classes are used to hold state and logic for the simpler function components containing UI elements. Although, with the introduction of hooks, which I will explain later, the lines have been blurred between each component type’s usefulness and functionality.

### Class Components

This type of component is declared using the class keyword and is extended using React.Component. Doing so will give access to all of React’s features, including a local state object and lifecycle methods (componentDidMount, componentWillUnmount, etc.).

In the pattern I briefly described above, these class components are used as containers for simpler UI components. The class will hold local state and methods to update said state that can be passed down to the presentation components as props.

```javascript
class Post extends React.Component {
  render () {
    <>
      <h1>Class Vs Function Components</h1>
      ....
    </>
  }
}
```

### Function Components

On the simpler side are the function components, declared using the function keyword or the ES6 arrow syntax, with a ‘props’ object as it’s only argument. They receive props from their parent component and render elements. Without being connected to React, the possibilities for these functions are limited and many choose to stick with classes to save time converting them later if state or another need arises.

That is until February 2019, when React v16.8.0 was released. This release gave function components a new feature called hooks. These functions granted by React (useState, useEffect, etc) gave function components the same access that class components have, allowing for much greater flexibility, rather than strict UI rendering.

```javascript
const Post = props => {
  return (
    <>
      <h1>Class Vs Function Components</h1>
      ....
    </>
  )
}
```

###Two hook explanations with example below:
**useState**
This function is how function components can create/update state. It is assigned using array destructuring to create variables for the state itself and a function to set the state.
```javascript
const [state, setState] = useState(*default state value*)
```

**useEffect**
The second example is where data fetching, subscriptions or manually changing the DOM would be located. These operations are called ‘side effects’, because they can affect other components and cannot be completed during rendering. This single function has the same purpose as all three of the class lifecycle methods componentDidMount, componentDidUpdate and componentWillUnmount.

```javascript
const Posts = props => {
  const [posts, setPosts] = useState([])

  useEffect(() => {
    fetch('/posts')
      .then(resp => json.())
      .then(post => setPosts(posts))
  }, [posts])

  return (
    <>
      {
        posts.map.....
      }
    </>
  )
}
```

### Conclusion
A couple of my favorite reasons for using function components are simplicity and readability.  In diving deeper into hooks during my research for this post, my ideas have only become more concrete. It seems that function components are where React will be heading. Though nothing has been deprecated and classes will be fully supported going forward, I feel the majority of reasons function components wouldn’t be used has been answered with hooks. I look forward to continuing my exploration of Hooks, React, Javascript and coding in general.

Thank you for reading.