# 아이템 24

# 멤버 클래스는 되도록 static으로 만들라

## 중첩 클래스 4가지 종류

1. **정적 멤버 클래스**
2. **비정적 멤버 클래스**
3. **익명 클래스**
4. **지역 클래스** 

## 1. 정적 멤버 클래스

### 특징

- static 키워드가 붙음
- 바깥 클래스의 인스턴스와 독립적
- 바깥 클래스의 static private 멤버에 접근 가능

```java
public class Calculator {

    public static enum Operation {
        PLUS {
            public double apply(double x, double y) { return x + y; }
        },
        MINUS {
            public double apply(double x, double y) { return x - y; }
        },
        MULTIPLY {
            public double apply(double x, double y) { return x * y; }
        },
        DIVIDE {
            public double apply(double x, double y) { return x / y; }
        };
        public abstract double apply(double x, double y);

    }
}

public static void main(String[] args) {
    Calculator.Operation plus = Calculator.Operation.MINUS; // 이 때 Calculator 인스턴스 생성 x
    double apply = plus.apply(1, 2);

    System.out.println(apply);
}
```

## 2. 비정적 멤버 클래스

### 특징

- static 키워드가 **없음**
- 바깥 클래스의 모든 멤버에 접근 가능
- 바깥클래스명.this로 바깥 인스턴스 참조 가능, 그냥 아무것도 안 붙여도 가능

```java
public class MusicPlayer {

    private List<String> playlist = new ArrayList<>();
    private int currentIndex = 0;
    private boolean isPlaying = false;
    private String playerName;

    public MusicPlayer(String playerName) {
        this.playerName = playerName;
    }

    public void addSong(String song) {
        playlist.add(song);
    }

    public void play() {
        isPlaying = true;
        System.out.println("재생 중: " + playlist.get(currentIndex));
    }

    public class PlaybackController {
        public void next() {
            if (currentIndex < playlist.size() - 1) {
                currentIndex++; // 내부 클래스에서 외부클래스의 필드를 접근 후 수정
                if (isPlaying) {
                    System.out.println("[" + playerName + "] 다음 곡: " +
                            playlist.get(currentIndex));
                }
            }
        }

        public void previous() {
            if (currentIndex > 0) {
                currentIndex--;
                if (isPlaying) {
                    System.out.println("[" + playerName + "] 이전 곡: " +
                            playlist.get(currentIndex));
                }
            }
        }

        public String getCurrentSong() {
            return playlist.get(currentIndex);
        }

        public boolean isPlaying() {
            return MusicPlayer.this.isPlaying;
        }

        public int getRemainingCount() {
            return playlist.size() - currentIndex - 1;
        }
    }

    public PlaybackController getController() {
        return new PlaybackController();
    }

    public static class Genre {
        public static final String POP = "팝";
        public static final String ROCK = "록";
        public static final String JAZZ = "재즈";
        public static final String CLASSIC = "클래식";

        private final String name;

        public Genre(String name) {
            this.name = name;
        }

        public String getName() {
            return name;
        }
    }
}

-----------------

public static void main(String[] args) {
    MusicPlayer player = new MusicPlayer("내 플레이어");
    player.addSong("Shape of You");
    player.addSong("Blinding Lights");
    player.addSong("Dynamite");
    player.play();

    MusicPlayer.PlaybackController controller = player.getController();
    controller.next();
    controller.next();
    System.out.println("현재 곡: " + controller.getCurrentSong());
    System.out.println("남은 곡: " + controller.getRemainingCount());

    MusicPlayer.Genre genre = new MusicPlayer.Genre(MusicPlayer.Genre.POP);

}

재생 중: Shape of You
[내 플레이어] 다음 곡: Blinding Lights
[내 플레이어] 다음 곡: Dynamite
현재 곡: Dynamite
남은 곡: 0
```

## **3-1. 비정적 vs 정적 멤버 클래스 비교(메모리 누수)**

```java
public class OuterClass {
    private byte[] data = new byte[1024 * 1024]; // 1MB
    
    class InnerClass {
        public void doSomething() {
            System.out.println("gggg");
        }
    }
    
    public InnerClass createInner() {
        return new InnerClass();
    }
}

List<OuterClass.InnerClass> list = new ArrayList<>();

for (int i = 0; i < 1000; i++) {
    OuterClass outer = new OuterClass(); // 계속 OuterClass가 만들어져서 메모리 누수
  list.add(outer.createInner());

// outer는 더 이상 필요 없지만,
// InnerClass가 참조를 가지고 있어서 GC되지 않음
```

```java
public class OuterClass {
    private byte[] data = new byte[1024 * 1024]; // 1MB
    
    // 정적 멤버 클래스
    static class InnerClass {
        public void doSomething() {
            System.out.println("aaaa");
        }
    }
    
    public static InnerClass createInner() {
        return new InnerClass();
    }
}

List<OuterClass.InnerClass> list = new ArrayList<>();

for (int i = 0; i < 1000; i++) {
    OuterClass outer = new OuterClass(); // 1MB 할당
    list.add(OuterClass.createInner());
    
    // InnerClass가 참조하지 않으므로 outer는 바로 GC됨 
}
```

## 3. 익명 클래스

이름이 없는 클래스로, 선언과 동시에 인스턴스가 생성

```java
button.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent e) {
        System.out.println("aaaa");
    }
});

==========

button.addActionListener(e -> System.out.println("aaaa"));
```

## 4. 지역 클래스

메서드 내부에 선언하는 클래스

```java
public class OuterClass {
    public void someMethod() {
        final String localVar = "지역 변수";
        
        **class LocalClass {
            public void print() {
                System.out.println(localVar);
                System.out.println(OuterClass.this.toString());
            }
        }
        
        LocalClass local = new LocalClass();
        local.print();
    }
}
```

## 사용 기준

- 바깥 클래스의 필드, 메서드를 사용하나?
    
    O → non staic
    
    X → static
    
- 한 메서드 내에서만 사용하나?
    
    O → 지역 or 익명 클래스
    
    X → 일반 멤버 클래스
    

**바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여라**

- 메모리 절약
- 메모리 누수 방지
- 명확한 의도 표현