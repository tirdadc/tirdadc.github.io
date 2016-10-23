---
title: "React.js: Firefox issue with onBlur"
date: 2015-06-11 07:38:30 -0400
comments: true
categories: [React.js, Firefox]
---

This started as a simple enough component to build in [React.js](https://facebook.github.io/react/): a text block that becomes an editable input when you click on it and reverts to text on the blur event.

``` javascript
var InputField = React.createClass({
  getInitialState: function() {
    return {
      value:  this.props.value,
      active: false
    }
  },

  updateValue: function(e) {
    e.preventDefault();
    this.setState({
      value: e.target.value
    });
  },

  handleClick: function(e) {
    this.setState({
      active: true
    });
  },

  onBlur: function(e) {
    this.setState({
      active: false
    });
  },

  render: function() {
    if (this.state.active) {
      return (
        <input
          type='text'
          value={this.state.value}
          onChange={this.updateValue}
          autoFocus={true}
          onBlur={this.onBlur} />;
    }
    else {
      return (
        <div>
          <span onClick={this.handleClick}>{this.state.value}</span>
        </div>
      )
    }
  }
})

React.render(<InputField value="click me to edit me" />, document.getElementById('container'));

```

This works perfectly fine in Chrome and Safari, but due to some [potential bug](https://www.google.com/search?q=firefox+onblur+bug), Firefox triggers the onBlur on the initial click on the text, reverting the active flag to false and preventing the input from ever being rendered.

This can be fixed by adding this to onBlur:

``` javascript
onBlur: function(e) {
  // Firefox issue
  if (e.nativeEvent.explicitOriginalTarget &&
      e.nativeEvent.explicitOriginalTarget == e.nativeEvent.originalTarget) {
    return;
  }
  this.setState({
    active: false
  });
}
```

JSFiddle: http://jsfiddle.net/gmk316qd/1/
