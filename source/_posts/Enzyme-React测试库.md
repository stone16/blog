---
title: Enzyme -- React测试库
date: 2020-05-18 16:06:42
categories: FrontEnd
tags:
    - Enzyme
top:
---
这是前端用于对React的组件进行测试的一个工具类，我们可以使用这个工具来遍历，控制，以及一定程度上的模拟运行时输出。我们主要是将该工具类和Jest一起使用，写我们的react组件的单元测试们。

# 1. Shalow Rendering 

用于以单个组件为单元来进行测试，然后确保你的测试不会依赖于子组件的状态。从Enzyme v3开始，shallow API会call React生命周期方法了，譬如`componentDidUpdate`和`componentDidMount`

    import { shallow } from 'enzyme';
    import sinon from 'sinon';
    import Foo from './Foo';

    describe('<MyComponent />', () => {
      it('renders three <Foo /> components', () => {
        const wrapper = shallow(<MyComponent />);
        expect(wrapper.find(Foo)).to.have.lengthOf(3);
      });

      it('renders an `.icon-star`', () => {
        const wrapper = shallow(<MyComponent />);
        expect(wrapper.find('.icon-star')).to.have.lengthOf(1);
      });

      it('renders children when passed in', () => {
        const wrapper = shallow((
          <MyComponent>
            <div className="unique" />
          </MyComponent>
        ));
        expect(wrapper.contains(<div className="unique" />)).to.equal(true);
      });

      it('simulates click events', () => {
        const onButtonClick = sinon.spy();
        const wrapper = shallow(<Foo onButtonClick={onButtonClick} />);
        wrapper.find('button').simulate('click');
        expect(onButtonClick).to.have.property('callCount', 1);
      });
    });
 
[API Reference](https://enzymejs.github.io/enzyme/docs/api/shallow.html)
# 2. Full Dom Rendering 

这种测试方式在你需要和DOM API进行交互，或者需要测试在更高次位的组件的时候非常有用。

需要运行在浏览器环境当中，如果无法运行在真实的浏览器当中，那我们就需要依赖于`mount`指令，在指令之下，是调用了一个叫做jsdom的包，完全使用JavaScript实现了一个浏览器。

值得注意的是，full dom rendering是真实的将当前组件渲染到DOM树当中，这也意味着如果用的是同一棵DOM树，那么你做的改动很可能会影响其他的测试，这点是值得我们注意的。


    import { mount } from 'enzyme';
    import sinon from 'sinon';
    import Foo from './Foo';

    describe('<Foo />', () => {
      it('calls componentDidMount', () => {
        sinon.spy(Foo.prototype, 'componentDidMount');
        const wrapper = mount(<Foo />);
        expect(Foo.prototype.componentDidMount).to.have.property('callCount', 1);
      });

      it('allows us to set props', () => {
        const wrapper = mount(<Foo bar="baz" />);
        expect(wrapper.props().bar).to.equal('baz');
        wrapper.setProps({ bar: 'foo' });
        expect(wrapper.props().bar).to.equal('foo');
      });

      it('simulates click events', () => {
        const onButtonClick = sinon.spy();
        const wrapper = mount((
          <Foo onButtonClick={onButtonClick} />
        ));
        wrapper.find('button').simulate('click');
        expect(onButtonClick).to.have.property('callCount', 1);
      });
    });

[API Reference](https://enzymejs.github.io/enzyme/docs/api/mount.html)

# 3. Static Rendering 

Render 使用的是Cheerio这个HTML转化库，用于从我们的React树来生成HTML，然后分析HTML的整个架构。

    import React from 'react';
    import { render } from 'enzyme';
    import PropTypes from 'prop-types';

    describe('<Foo />', () => {
      it('renders three `.foo-bar`s', () => {
        const wrapper = render(<Foo />);
        expect(wrapper.find('.foo-bar')).to.have.lengthOf(3);
      });

      it('rendered the title', () => {
        const wrapper = render(<Foo title="unique" />);
        expect(wrapper.text()).to.contain('unique');
      });

      it('renders a div', () => {
        const wrapper = render(<div className="myClass" />);
        expect(wrapper.html()).to.contain('div');
      });

      it('can pass in context', () => {
        function SimpleComponent(props, context) {
          const { name } = context;
          return <div>{name}</div>;
        }
        SimpleComponent.contextTypes = {
          name: PropTypes.string,
        };

        const context = { name: 'foo' };
        const wrapper = render(<SimpleComponent />, { context });
        expect(wrapper.text()).to.equal('foo');
      });
    });
# Reference
1. https://enzymejs.github.io/enzyme/