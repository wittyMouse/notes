### 数据存储
1. 寄存器（Registers）
2. 栈内存（Stack）
> 创建程序时，Java 系统必须知道栈内保存的所有项的生命周期。这种约束限制了程序的灵活性。因此，虽然在栈内存上存在一些 Java 数据（如对象引用），但 Java 对象本身的数据却是保存在堆内存的。
3. 堆内存（Heap）
4. 常量存储（Constant storage）
5. 非RAM存储（Non-RAM storage）
    1. 序列化对象
    2. 持久化对象

| 基本类型 | 大小 | 最小值 | 最大值 | 包装类型 |
| --- | --- | --- | --- | --- |
| boolean | — | — | — | Boolean |
| char | 16 bits | Unicode 0 | Unicode 216 -1 | Character |
| byte | 8 bits | -128 | +127 | Byte |
| short | 16 bits | - 215 | + 215 -1 | Short |
| int | 32 bits | - 231 | + 231 -1 | Integer |
| long | 64 bits | - 263 | + 263 -1 | Long |
| float | 32 bits | IEEE754 | IEEE754 | Float |
| double | 64 bits | IEEE754 | IEEE754 | Double |
| void | — | — | — | void |

### 二叉树的遍历

二叉树的深度优先遍历可细分为前序遍历、中序遍历、后序遍历，这三种遍历可以用递归实现，也可使用非递归实现。

前序遍历：根节点 -> 左子树 -> 右子树（根 -> 左 -> 右）

中序遍历：左子树 -> 根节点 -> 右子树（左 -> 根 -> 右）

后序遍历：左子树 -> 右子树 -> 根节点（左 -> 右 -> 根）

### 2020.07.22
接口也可以作为一种类型

#### 设计模式
1. 工厂模式
```java
public interface Bird {
	public void skill();
}

public class Swallow implements Bird {
	@Override
	public void skill() {
		System.out.println("I can fly!");
	}
}

public class Magpie implements Bird {
	@Override
	public void skill() {
		System.out.println("I can twitter!");
	}
}

public class BirdFactory {
	public static Swallow getSwallow() {
		return new Swallow();
	}
	
	public static Magpie getMagpie() {
		return new Magpie();
	}
}

public class BirdFactoryDemo {
	public static void main(String[] args) {
		Swallow swallow = BirdFactory.getSwallow();
		swallow.skill(); // I can fly!
		
		Magpie magpie = BirdFactory.getMagpie();
		magpie.skill(); // I can twitter!
	}
}
```

>术语：Producer（生产者）、Consumer（消费者）

2. 抽象工厂模式
```java
public interface Bird {
	public void skill();
}

public class Swallow implements Bird {
	@Override
	public void skill() {
		System.out.println("I can fly!");
	}
}

public class Magpie implements Bird {
	@Override
	public void skill() {
		System.out.println("I can twitter!");
	}
}

public interface Fish {
	public void skill();
}

public class Crocodile implements Fish {
	@Override
	public void skill() {
		System.out.println("I can crawl!");
	}
}

public class Whale implements Fish {
	@Override
	public void skill() {
		System.out.println("I can blow!");
	}
}

public abstract class AbstractFactory {
	public abstract Bird getBird(BirdEnum birdEnum);
	public abstract Fish getFish(FishEnum fishEnum);
}

public enum BirdEnum {
	SWALLOW, MAGPIE;
}

public class BirdFactory extends AbstractFactory {
	@Override
	public Bird getBird(BirdEnum birdEnum) {
		Bird bird = null;
		switch (birdEnum) {
		case SWALLOW:
			bird = new Swallow();
			break;
		case MAGPIE:
			bird = new Magpie();
			break;
		default:
			break;
		}
		return bird;
	}
	
	@Override
	public Fish getFish(FishEnum fish) {
		return null;
	}
}

public enum FishEnum {
	CROCODILE, WHALE;
}

public class FishFactory extends AbstractFactory {
	@Override
	public Bird getBird(BirdEnum Bird) {
		return null;
	}
	
	@Override
	public Fish getFish(FishEnum fishEnum) {
		Fish fish = null;
		switch (fishEnum) {
		case CROCODILE:
			fish = new Crocodile();
			break;
		case WHALE:
			fish = new Whale();
			break;
		default:
			break;
		}
		return fish;
	}
}

public enum AnimalEnum {
	BIRD, FISH;
}

public class AbstractFactoryProducer {
	public static AbstractFactory getFactory(AnimalEnum animalEnum) {
		AbstractFactory abstractFactory = null;
		switch (animalEnum) {
		case BIRD:
			abstractFactory = new BirdFactory();
			break;
		case FISH:
			abstractFactory = new FishFactory();
			break;
		default:
			break;
		}
		return abstractFactory;
	}
}

public class AbstractFactoryPatternDemo {
	public static void main(String[] args) {
		AbstractFactory birdFactory = AbstractFactoryProducer.getFactory(AnimalEnum.BIRD);
		AbstractFactory fishFactory = AbstractFactoryProducer.getFactory(AnimalEnum.FISH);
		
		Bird swallow = birdFactory.getBird(BirdEnum.SWALLOW);
		Bird magpie = birdFactory.getBird(BirdEnum.MAGPIE);
		swallow.skill(); // I can fly!
		magpie.skill(); // I can twitter!
		
		Fish crocodile = fishFactory.getFish(FishEnum.CROCODILE);
		Fish whale = fishFactory.getFish(FishEnum.WHALE);
		crocodile.skill(); // I can crawl!
		whale.skill(); // I can blow!
	}
}
```
