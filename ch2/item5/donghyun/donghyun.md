# 아이템 5

## 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

**정적 유틸리티를 잘못 사용한 사례들**

```java
public class SpellChecker {
	private static final Lexicon dictionary = new EnglishDictionary();
	
	private SpellChecker() {}
	
	public static boolean isValid(String word){}
	public static List<String> suggestions(String typo){}
}

or

public class SpellChecker {
	private static final Lexicon dictionary = new EnglishDictionary();
	
	public static final SpellChecker INSTANCE = new SpellChecker();
	
	private SpellChecker() {}
	
	public static boolean isValid(String word){}
	public static List<String> suggestions(String typo){}
}

-------

문제점

SpellChecker checker = SpellChecker.INSTANCE; 
checker.isValid("가나다라마바사"); 
// 지금 EnglishDictionary가 생성되어 있으므로 다른 언어는 검사 못함, 즉 유연함이 부족  
```

### **해결책 :** 의존 객체 주입

**생성자**

```java
public class SpellChecker {
	private final Lexicon dictionary;
	
	public SpellChecker(Lexicon dictionary) { // 생성자 주입
		this.dictionary = dictionary;
	}
	
	public static boolean isValid(String word){}
	public static List<String> suggestions(String typo){}
}

-> 사용
Lexicon의 자식으로 KoreanDictionary, EnglishDictionary가 있다고 가정

1. 한국어 문법 검사하고 싶을 경우
SpellChecker checker = new SpellChecker(new KoreanDictionary());
checker.isValid("가나다");

2. 영어 문법 검사하고 싶을 경우
SpellChecker checker = new SpellChecker(new EnglishDictionary());
checker.isValid("abcde");

```

**정적 팩토리 주입**

```java
public class SpellChecker {
    private final Lexicon dictionary;
    
    private SpellChecker(Lexicon dictionary) {
        this.dictionary = dictionary;
    }
    
    public static SpellChecker create(Lexicon dictionary) {
        return new SpellChecker(dictionary);
    }
}
```

**빌더 패턴**

```java
public class SpellChecker {
    private final Lexicon dictionary
    
    private SpellChecker(Builder builder) {
        this.dictionary= builder.dictionary;
    }
    
    public static class Builder {
        private Lexicon dictionary;
        
        public Builder createDictionary(Lexicon dict) {
            this.dictionary= dict;
            return this;
        }
        
        public SpellChecker build() {
            return new SpellChecker(this);
        }
    }
}

// 사용 예시
SpellChecker checker = new SpellChecker.Builder()
    .dictionary(new EnglishDictionary())
    .build();
```

**spring 의존성 주입**

```java
@Configuration
public class DictionaryConfig {

    @Bean
    @Qualifier("english")
    public Lexicon englishDictionary() {
        return new EnglishDictionary();
    }

    @Bean
    @Qualifier("korean")
    public Lexicon koreanDictionary() {
        return new KoreanDictionary();
    }
}

---

@Component
public class SpellCheckService {

    private final Map<String, Lexicon> dictionaries;

    @Autowired
    public SpellCheckService(
            @Qualifier("english") Lexicon englishDict,
            @Qualifier("korean") Lexicon koreanDict
    ) {

        this.dictionaries = Map.of(
                "en", englishDict,
                "ko", koreanDict
        );
    }

    public SpellChecker getChecker(String language) {
        Lexicon dictionary = dictionaries.get(language); // "en"이면 영어사전
        return new SpellChecker(dictionary);
    }
}

---

@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = {DictionaryConfig.class, SpellCheckService.class})
public class SpellCheckerTest {

    @Autowired
    private SpellCheckService spellCheckService;

    @Test
    public void test() {
        SpellChecker enChecker = spellCheckService.getChecker("ko");
        System.out.println(enChecker.getDictionary().getClass());
    }
}

---

결과 : class hello.proxy.effective.KoreanDictionary
```