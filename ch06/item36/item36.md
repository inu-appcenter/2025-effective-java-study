# 아이템 36

# 아이템 36: 비트 필드 대신 EnumSet을 사용하라

## 나쁜 예시

```java
public class Text {
    public static final int STYLE_BOLD          = 1 << 0; // 1
    public static final int STYLE_ITALIC        = 1 << 1; // 2
    public static final int STYLE_UNDERLINE     = 1 << 2; // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8

    public void applyStyles(int styles) {
        ...
    }
}

int styles = Text.STYLE_BOLD | Text.STYLE_ITALIC;
text.applyStyles(styles); // 3 (BOLD + ITALIC)
```

1. 타입 안전성x
    
    매개변수를 전혀 관련 없는 값도 전달 가능
    
2. 출력, 해석이 어려움
    
    위처럼 3이면 어떤 조합인지 알기가 어려움
    
3. 확상성 부족
    
    더 많은 상수를 추가할려면 타입 int → long으로 변경
    
4. 가독성 낮음
5. 반복문 사용 불가능(두 개의 Text가 들어온다고 해도 int로 바꾸기 때문)

## EnumSet

```java
public class Text {

    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

    public void applyStyles(Set<Style> styles) {
       for (Style style : styles) {
            System.out.println(style + " 스타일 적용");
        }
    }
}

text.applyStyles(EnumSet.of(Text.Style.BOLD, Text.Style.UNDERLINE, Text.Style.STRIKETHROUGH));
BOLD 스타일 적용
UNDERLINE 스타일 적용
STRIKETHROUGH 스타일 적용
```

1. 내부적으로 long(64) 기반 비트 벡터로 구현
2. SET 인터페이스 완전 구현체이므로 다양한 메서드 사용가능
    
    add, remove, contains, retainAll …..
    
3. 정적 팩터리 메서드 제송
    
    EnumSet.of() : 필요한 것만 직접 지정(EnumSet.of(Style.BOLD, Style.ITALIC);EnumSet.of(Style.BOLD, Style.ITALIC);)
    
    allOf() : 해당 Enum의 모든 값(EnumSet.allOf(Style.class))
    
    noneOf(): 비워두기(EnumSet.noneOf(Style.class))
    

## 결론

EnumSet

타입 보장, 가독성, 반복문 가능, 확장성(Enum 갯수만큼 가능), 성능 비슷