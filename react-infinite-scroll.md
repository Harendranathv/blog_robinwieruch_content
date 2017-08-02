+++
title = "Infinite Scroll in React - Build a powerful Component (Part II)"
description = "The series of React tutorials focuses on building a complex yet elegant and powerful React component. It attempts to go beyond the fundamentals in React. This part introduces infinite scroll in React..."
date = "2017-06-07T13:50:46+02:00"
tags = ["react"]
categories = ["React"]
keyword = "react infinite scroll"
news_keywords = ["react infinite scroll"]
hashtag = "#ReactJs"
contribute = "react-infinite-scroll.md"
card = "img/posts/react-infinite-scroll/banner_640.jpg"
banner = "img/posts/react-infinite-scroll/banner.jpg"
headline = "Infinite Scroll in React - Build a powerful Component (Part II)"

summary = "The series of React tutorials focuses on building a complex yet elegant and powerful React component. It attempts to go beyond the fundamentals in React. This part introduces infinite scroll in React. You will use higher order components to opt-in these functionalities in an elegant way."
+++

{{% pin_it_image "react infinite scroll" "img/posts/react-infinite-scroll/banner.jpg" "is-src-set" %}}

The following React tutorial builds up on [Paginated List in React - Build a powerful Component (Part I)](https://www.robinwieruch.de/react-paginated-list). The series of tutorials goes beyond the basic React components that you encounter in other React tutorials.

This part of the series will show you how to build an infinite scroll in React. So far, your List component is able to opt-in two functionalities: showing a loading indicator and fetching more list items by using a More button. While the More button fetches manually more items, the infinite scroll should fetch more items once the user scrolled to the end of the list.

In this part of the series, both functionalities, the manual and automatic retrieval, should be opt-in exclusively. In the third part of the series you will learn how to combine both enhancements in one advanced List component with error and fallback handling.

* [Paginated List in React - Build a powerful Component (Part I)](https://www.robinwieruch.de/react-paginated-list)
* **Infinite Scroll in React - Build a powerful Component (Part II)**
* [Advanced List in React - Build a powerful Component (Part III)](https://www.robinwieruch.de/react-advanced-list-component)

{{% chapter_header "Infinite Scroll in React" "infinite-scroll-react" %}}

The infinite scroll requires you to use lifecycle methods in the List component. These lifecycle methods are used to register the event listeners that trigger on scrolling. First, let's refactor the List component from a functional stateless component to a React ES6 class component. Otherwise we would not be able to access the lifecycle methods.

{{< highlight javascript "hl_lines=10 11 12 13 14 15 16 17 18 19 20 21 22" >}}
// functional stateless component
const List = ({ list }) =>
  <div className="list">
    {list.map(item => <div className="list-row" key={item.objectID}>
      <a href={item.url}>{item.title}</a>
    </div>)}
  </div>

// React ES6 class component
class List extends React.Component {
  render() {
    const { list } = this.props;
    return (
      <div className="list">
        {list.map(item => <div className="list-row" key={item.objectID}>
          <a href={item.url}>{item.title}</a>
        </div>)}
      </div>
    );
  };
}
{{< /highlight >}}

Now it's about implementing the functionality that the List fetches more paginated lists on scroll.

{{< highlight javascript "hl_lines=2 3 4 6 7 8 10 11 12 13 14 15 16 17" >}}
class List extends React.Component {
  componentDidMount() {
    window.addEventListener('scroll', this.onScroll, false);
  }

  componentWillUnmount() {
    window.removeEventListener('scroll', this.onScroll, false);
  }

  onScroll = () => {
    if (
      (window.innerHeight + window.scrollY) >= (document.body.offsetHeight - 500) &&
      this.props.list.length
    ) {
      this.props.onPaginatedSearch();
    }
  }

  render() {
    ...
  };
}
{{< /highlight >}}

There are two registered event listeners now. First, when the component mounted, the `onScroll()` method is registered as callback for the `scroll` event. Second, the same method gets unregistered when the component unmounts.

The `onScroll()` class method itself is responsible to execute the `onPaginatedSearch()` method that fetches the next page, the next subset, of the whole list. But it comes with two conditions: First, it only executes when the user reached the bottom of the page. Second, it executes only when there is already an initial list.

{{% chapter_header "Infinite Scroll as Higher Order Component in React" "infinite-scroll-higher-order-component" %}}

Again, as for the paginated list in the first part of the tutorial series, you can extract the functionality to a higher order component. You can already see all the parts of the List component that could move into the HOC: all those that you have added to the List component in the last step.

If you are not familiar with higher order components, as in the first part of the series, I can recommend to read [the gentle introduction to higher order components](https://www.robinwieruch.de/gentle-introduction-higher-order-components/).

{{< highlight javascript >}}
const withInfiniteScroll = (Component) =>
  class WithInfiniteScroll extends React.Component {
    componentDidMount() {
      window.addEventListener('scroll', this.onScroll, false);
    }

    componentWillUnmount() {
      window.removeEventListener('scroll', this.onScroll, false);
    }

    onScroll = () => {
      if (
        (window.innerHeight + window.scrollY) >= (document.body.offsetHeight - 500) &&
        this.props.list.length
      ) {
        this.props.onPaginatedSearch();
      }
    }

    render() {
      return <Component {...this.props} />;
    }
  }
{{< /highlight >}}

The List component becomes simple again. In addition, it doesn't need lifecycle methods anymore and can be refactored to a functional stateless component again.

{{< highlight javascript "hl_lines=1 2 3 4 5 6 7" >}}
const List = ({ list }) =>
  <div className="list">
    {list.map(item => <div className="list-row" key={item.objectID}>
      <a href={item.url}>{item.title}</a>
    </div>)}
  </div>
{{< /highlight >}}

Finally you can use the automatic infinite scroll instead of the manual paginated list.

{{< highlight javascript "hl_lines=15 19 27 28 29" >}}
class App extends React.Component {

  ...

  render() {
    return (
      <div>
        <h1>Search Hacker News</h1>

        <form type="submit" onSubmit={this.onInitialSearch}>
          <input type="text" ref={node => this.input = node} />
          <button type="submit">Search</button>
        </form>

        <ListWithLoadingWithInfinite
          list={this.state.hits}
          page={this.state.page}
          onPaginatedSearch={this.onPaginatedSearch}
        />
      </div>
    );
  }
}

...

const ListWithLoadingWithInfinite = compose(
  // withPaginated,
  withInfiniteScroll,
  withLoading,
)(List);
{{< /highlight >}}

Now, the two HOCs, paginated list and infinite scroll, can be opted-in exclusively to substitute the functionalities of manual and automatic retrieval of the next page of the list. Both can be combined with the loading indicator HOC though.

{{% chapter_header "Too many Request on Infinite Scroll" "too-many-request-react" %}}

There is one flaw in the infinite scroll higher order component. It executes too often, once the user reached the bottom of the page. But it should only execute once, wait for the result, and then be allowed to trigger again when the user reaches the bottom of the page.

{{< highlight javascript "hl_lines=9" >}}
const withInfiniteScroll = (Component) =>
  class WithInfiniteScroll extends React.Component {
    ...

    onScroll = () => {
      if (
        (window.innerHeight + window.scrollY) >= (document.body.offsetHeight - 500) &&
        this.props.list.length &&
        !this.props.isLoading
      ) {
        this.props.onPaginatedSearch();
      }
    }

    ...
  }
{{< /highlight >}}

Now the loading state prevents too many requests. Only when there is no pending request, the scroll event will trigger.

<hr class="section-divider">

The upcoming and last part of this series will show you how to combine both functionalities, the paginated list and the infinite scroll, to make it a great user experience. A little hint: one of the two can be used as fallback when there was an erroneous request. Other platforms, such as Twitter and Pocket, are using this approach for an improved UX.

You can continue with the second part of the React tutorial series: [Advanced List in React - Build a powerful Component (Part III)](https://www.robinwieruch.de/react-advanced-list-component).