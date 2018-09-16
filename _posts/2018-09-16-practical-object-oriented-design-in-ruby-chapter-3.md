### 3장 의존성 관리하기

------

잘 디자인된 객체는 하나의 책임만 지고 있기 때문에 객체가 복잡한 작업을 수행하기 위해서는 다른 객체와 협업을 해야한다. 서로 협업을 하려면 객체는 다른 객체에 대한 지식이 있어야 한다.이러한 지식이 의존성을 만들어낸다.



#### 3.1 의존성 이해하기

하나의 객체를 수정했을 때, 다른 객체들을 뒤따라 수정해야 한다면 이것은 의존적이다.

의존성이 있다는 것 자체가 객체를 수정하기 어렵고 연약하게 만든다.

~~~ruby
class Gear
  attr_accessor :chainring, :cog, :rim, :tire
  def initialize(chainring, cog, rim, tire)
    @chainring = chainring
    @cog = cog
    @rim = rim
    @tire = tire
  end

  def ratio
    chainring / cog.to_f
  end

  def gear_inches
    ratio * Wheel.new(rim, tire).diameter
  end
end

class Wheel
  attr_reader :rim, :tire

  def initialize(rim, tire)
    @rim = rim
    @tire = tire
  end

  def diameter
    (rim + (tire * 2))
  end

  def circumference
    diameter * Math::PI
  end
end

puts Gear.new(52, 11, 26, 1.5).gear_inches
~~~



##### 3.1.1 의존성이 있다는 것을 알기

- 다른 클래스의 이름. Gear는 Wheel이라는 이름의 클래스가 있다는 걸 알고 있다.
- 자기 자신을 제외한 다른 객체에게 전송할 메시지의 이름. Gear는 Wheel의 인스턴스가 diameter라는 메서드를 이해할 수 있다는 것을 알고 있다.
- 메시지가 필요로 하는 인자들. Gear는 Wheel.new를 위해 rim과 tire를 인자로 넘겨야 한다는 것을 알고 있다.
- 인자들을 전달하는 순서. Gear는 Wheel.new의 첫번째 인자가 rim이고 두번째 인자가 tire라는 것을 알고 있다.

이러한 의존성이 Gear클래스의 수정을 강제한다. 



##### 3.1.2 객체들 간의 결합

이런 의존성은 Gear를 Wheel에 결합(couple)시킨다. 두 객체가 강하게 결합할수록 Wheel을 수정하면 Gear도 같이 수정해야한다. 

##### 3.1.3 다른 의존성들

일반적인 의존성 이슈

- 메시지 연쇄(message chaining)

  하나의 객체가 다른 객체에 대해 알고 있는데 이 다른 객체가 무언가를 알고 있는 또 다른 객체에 대해 알고 있는 경우, 여러 개의 메시지가 여러 단계를 거쳐 연결되어(chained) 저 멀리 있는 객체의 행동을 실행시키려 하는 메시지 연쇄(message chaining)은 최종 목표까지 도달하기 위해 거쳐갔던 모든 객체와 메시지들 사이에도 의존성을 만들어낸다. 연쇄의 중간에 끼어 있던 어떤 객체가 변하더라도 첫번째까지 영향을 미친다.

- 테스트 코드가 갖는 의존성

  테스트는 코드를 참조하고 그런 의미에서 코드에 의존적이다. 코드와 지나치게 결합된 테스트는 코드의 핵심적인 내용은 바뀌지 않았는데도 불구하고 리팩토링때마다 테스트가 깨져간다. 테스트-코드 사이의 지나친 결합은 코드-코드 사이의 지나친 결합과 같은 결과는 낳는다.

#### 3.2 약하게 결합된 코드 작성하기

##### 3.2.1 의존성 주입하기

Gear클래스에서 gear_inches메서드는 Wheel 클래스를 명시적으로 참조하고 있다.

~~~ruby
class Gear
  attr_accessor :chainring, :cog, :wheel
  def initialize(chainring, cog, wheel=nil)
    @chainring = chainring
    @cog = cog
    @rim = rim
    @tire = tire
  end

  def gear_inches
    ratio * Wheel.new(rim, tire).diameter
  end
end
~~~

wheel 클래스의 이름이 바뀌면 Gear의 gear_inches 메서드도 함께 변경되어야 한다. Gear가 Wheel 클래스의 이름을 알고 있다면, Wheel의 이름이 바뀌었을때 Gear 코드의 한 부분이 수정되어야 한다. 클래스 이름이 변경되는 건 큰 문제는 아니지만,  문제는 Gear는 Wheel 이외의 다른 종류의 객체와 협업을 거부하고 있다.

중요한 것은 '객체의 클래스가 무엇인지'가 아니라 '우리가 전송하는 메시지가 무엇인지'이다.  Gear는 gear_inches를 계산하기 위해 diameter를 알고 있는 객체만 있으면 된다.

~~~ruby
class Gear
  attr_accessor :chainring, :cog, :wheel
  def initialize(chainring, cog, wheel=nil)
    @chainring = chainring
    @cog = cog
    @wheel = wheel
  end

  def ratio
    chainring / cog.to_f
  end

  def gear_inches
    ratio * wheel.diameter
  end
end
~~~

Gear는 @wheel객체가 Wheel 클래스의 인스턴스라는 것을 알지도 못하고, 관심도 없다. Gear가 알고 있는 것은 자기 자신이 diameter메서드에 반응할 줄 아는 객체를 가지고 있다는 것뿐이다.

이렇게 구현함으로써, Gear와 Wheel사이의 의존성이 없어지고, 다른 객체와도 협업이 가능해졌다. 이러한 기술을 의존성 주입(dependency injection)라고 한다.

##### 3.2.2 의존성 격리시키기

불필요한 의존성을 제거할 수 없는 경우라면, 의존성을 클래스 안에서 격리시켜 놓아야 한다. 

##### 인스턴스 생성을 격리시키기

Gear에 Wheel을 주입 할 수 없다면 Wheel인스턴스를 만드는 과정을 Gear 클래스 내부에 격리시켜 놓을 필요가 있다. 다음 두 개의 예시는 이런 접근법을 보여준다.

1.

1. ~~~ruby
   class Gear
     attr_accessor :chainring, :cog, :wheel
     def initialize(chainring, cog, rim, tire)
       @chainring = chainring
       @cog = cog
       @wheel = Wheel.new(rim, tire)
     end
   
     def gear_inches
       ratio * wheel.diameter
     end
   #...
   end
   
   ~~~

   Wheel 인스턴스를 생성하는 과정을 initialize로 옮기고, wheel에 대한 의존성을 더욱 뚜렷하게 드러낼 수 있게 되었다.

2. 

   ~~~ruby
   class Gear
     attr_accessor :chainring, :cog, :rim, :tire
     def initialize(chainring, cog, rim, tire)
       @chainring = chainring
       @cog = cog
       @rim = rim
       @tire = tire
     end
   
     def gear_inches
       ratio * wheel.diameter
     end
   
     def wheel
       @wheel ||= Wheel.new(rim, tire)
     end
   #...
   end
   
   ~~~

   wheel 메소드를 통해 새로운 Wheel 인스턴스를 만드는 방법이다. ||= 연산자를 통해 필요한 순간에 왔을때 Wheel인스턴스를 만든다.

의존성을 염두에 두고 의존성을 주입하는 코딩 습관을 들일때 클래스는 자연스럽게 덜 결합된 형태를 띈다.

##### 외부로 전송하는 메시지 중 위험한 것들을 격리시키기

외부로 전송되는 메시지란 '나 자신 아닌 객체에게 보내는 메시지'이다.예를 들어 gear_inches는 ratio와 wheel 메시지를 자기 자신에게 보내고 있지만, diameter메시지는 wheel에게 보내고 있다.

~~~ruby
def get_inches
  ratio * wheel.diameter
end
~~~

Gear가 wheel메서드에 반응한다는 사실에 의존하고 이 wheel이 diameter메서드에 반응한다는 사실에도 의존한다. 외부에 대한 의존성을 걷어내고 의존성을 클래스 내부의 메서드 속에 캡슐화시켜 놓으면 메서드를 수정해야 하는 상황도 줄일 수 있다.

~~~ruby
def gear_inches
   foo = som_intermediate_result * diameter
end

def diameter
    wheel.diameter
end
~~~

 wheel.diameter를 호출하는 곳이 많았다면, 처음부터 만들었겠지만, 차이점은 타이밍이다. 

만약 Wheel이 자신이 구현하고 있는 diameter 메서드의 이름과 시그너처를 바꾸더라도 Gear에게 미치는 영향은 wrapping method에 한정될 것이다.

#####3.2.3 인자 순서에 대한 의존성 제거하기

인자를 넘기는 작업은 종종 조금 더 감지하기 어려운 또 하나의 의존성을 만들어낸다.

~~~ruby
class Gear
  attr_accessor :chainring, :cog, :wheel
  def initialize(chainring, cog, wheel)
    @chainring = chainring
    @cog = cog
    @wheel = wheel
  end
#...
end

puts Gear.new(52, 11, Wheel.new(26, 1.5))
~~~

new 메서드를 전송하는 송신자는 Gear의 initialize 메서드에서 정의된 인자의 순서에 의존적이다. 초기단계에 인자들을 추가하고 제거하고 할 때, 순서에 의존적이면 곤란해진다.

##### 초기화 인자로 해시를 사용하기

~~~ruby
class Gear
  attr_accessor :chainring, :cog, :wheel
  def initialize(options)
    @chainring = options[:chainring]
    @cog = options[:cog]
    @wheel = options[:wheel]
  end
#...
end

puts Gear.new(
  chainring: 52,
  cog: 11,
  wheel: Wheel.new(26, 1.5)
)
~~~

인자들의 순서에 대한 의존성을 제거했다. 해시를 사용해서 순서에 대한 의존성을 없앨 수 있지만, 해시 키(key)의 이름에 의존하게 되었다.

꼭 필요한 인자가 몇개 있고 변경될 가능성이 있는, 추가적인 인자들이 있는 그런 일반적인 상황들이 있는 경우, 두가지 방법을 동시에 사용하는 편이 가장 효율적이다. 몇개의 고정된 이자를 받고 추가적인 인자들을 해시로 받으면 된다.

##### 기본값 사용하기

~~~ruby
class Gear
  attr_accessor :chainring, :cog, :wheel
  def initialize(options)
    @chainring = options[:chainring] || 40
    @cog = options[:cog] || 18
    @wheel = options[:wheel]
  end
#...
end
~~~

한가지 주의할 점은 || 연산자는 or 처럼 작동해서 Hash의 등록되지 않는 키에 대한 값을 요청하면 nil을 반환하기에 

~~~ruby
@bool = args[:boolean_thing] || true
~~~

의 경우, @bool값이 true가 되어버린다. 이경우에는 fetch 메소드를 쓰는 것이 좋다.

##### 멀티파라미터(Multiparameter)초기화를 고립시키기

메서드를 직접 수정할 수 없는 상황이면, 메서드를 직접 수정할 수 없기 때문에 순서가 고정된 인자들을 갖고 있는 메서드를 사용해야만 하는 상황도 있다. Gear가 특정 프레임워크의 한 부분이고 initialize 메서드가 순서가 정해진 인자들을 필요로 한다고 생각해보면, Gear 인스턴스를 생성하는 지점을 하나의 메서드로 감싸는 것을 통해 코드를 DRY하게 만들 수 있다.

~~~ruby
module::SomeFramework
  class Gear
    attr_accessor :chainring, :cog, :wheel
    def initialize(chainring, cog, wheel=nil)
      @chainring = chainring
      @cog = cog
      @wheel = wheel
    end
  end
  #...
end

# Wrap the interface to protect yourself from changes
module GearWrapper
  def self.gear(options)
    SomeFramework::Gear.new(
      options[:chainring],
      options[:cog],
      options[:wheel]
      )
  end
end

# So now you can do:
GearWrapper.gear(
  chainring: 52,
  cog: 11,
  wheel: Wheel.new(26, 1.5)
)
~~~



#### 3.3 의존성의 방향 관리하기

##### 3.3.1 의존성 방향 바꾸기

Gear는 Wheel이나 diameter에 의존했지만, 의존성을  반대로 설정할 수도 있었다. 

~~~ruby
class Gear
  attr_accessor :chainring, :cog
  def initialize(chainring, cog)
    @chainring = chainring
    @cog = cog
  end

  def gear_inches(diameter)
    ratio * diameter
  end

  def ratio
    chainring / cog.to_f
  end
end

class Wheel
  attr_reader :rim, :tire, :gear

  def initialize(rim, tire, chainring, cog)
    @rim = rim
    @tire = tire
    @gear = Gear.new(chainring, cog)
  end

  def diameter
    (rim + (tire * 2))
  end

  def gear_inches
    gear.gear_inches(diameter)
  end
end

p Wheel.new(26,1.5,52,11).gear_inches
~~~

#####3.3.2 의존성의 방향 결정하기

**자기 자신보다 덜 변하는 사람들에게 의존하라**고 말할 수 있다.

- 어떤 클래스는 다른 클래스에 비해 요구사항이 더 자주 바뀐다.
- 구체 클래스(concrete classes)는 추상 클래스(abstract classes)보다 수정해야 하는 경우가 빈번히 발생한다.
- 의존성이 높은 클래스를 변경하는 것은 코드의 여러 곳에 영향을 미친다.

##### 변경될 가능성이 얼마나 높은지 이해하기

우리가 작성하지 않았지만 사용하고 있는 코드 즉 우리가 의존하고 있는 루비의 베이스 클래스(Base classes)나 다른 프레임워크 코드도 바뀌지만, 우리가 작성하는 코드보다 바귈 가능성이 낮다. 따라서 우리는 베이스 클래스에 의존해도 괜찮다.

의존성의 방향은 다른 클래스와 비교해서 얼마나 변경되지 않는지에 따라 순위를 매겨볼 수 있다.

##### 구체적인 것과 추상적인 것을 인지하기

##### 의존성이 높은 클래스 만들지 않기

의존성이 높은 클래스는 작은 수정으로 애플리케이션 전체를 뜯어 고치게 만들 수 있기 때문에 의존성이 높은 클래스는 만들지 않아야된다.

##### 문제가 되는 의존성 찾아내기

중요한 디자인 결정을 내려하는 순간은 수정을 해야 할 수도 있는 가능성이 여러 의존성과 만나는 지점에서 발생한다. 이 둘 사이의 어떤 조합은 애플리케이션에 도움을 주지만 다른 조합은 치명적인 결과를 낳을 수 있다. 

A 영역에는 추상 클래스나 인터페이스가 속한다. 클래스에 의존하고 있는 객체, 즉 딸린 객체가 많은 클래스이다. A 영역은 추상화가 되었기 때문에 A영역에 속하게 된 것이다. 추상화 자체가 클래스들을 안정적으로 만들어주고 의존성을 안전하게 관리 할 수 있게 해준다.

C 영역은 변경될 가능성이 높지만 클래스에 의존하고 있는 달린 객체가 적은 클래스들로 채워져있다. 변경될 가능성도 높지만 의존하고 있는 클래스들이 별로 없기 때문에 크게 문제 되지 않는다.

B 영역은 변할 일도 없고 클래스에 의존하고 있는 딸린 객체들도 적다.

A,B,C 영역은 코드가 있어도 괜찮은 역역이지만 D 영역은 위험 영역이다. D 영역의 클래스들은 애플리케이션의 건강 상태를 드러내는 지표이다. 이 클래스들 때문에 애플리케이션을 수정하기 어렵다.  

어떤 클래스가 어느 영역에 속하는지 파악하기 쉽지 않다.





 