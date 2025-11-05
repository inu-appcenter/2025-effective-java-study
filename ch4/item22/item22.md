# item22

## 🏁 인터페이스는 타입을 정의하는 용도로만 사용하라

---

클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에 얘기해주는 것이다.

❌ 상수 인터페이스 (안티패턴)

```jsx
public interface PhysicaKonstants {
// 아보가드로 수 (1/몰)
static final double AVOGADROS_NUMBER = 6.022_140_857e23;
// 볼츠만 상수 (J/K)
static final double BOLTZMW_CONSTANT = 1.380_648_52e-23;
// 전자 질량 (kg)
static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```

→ 이는 잘못됐다.

→ 클래스 내부에서 사용하는 상수는 외부 인터페이스가 아니라, 내부 구현이다.

→ 즉 이를 구현하는 클래스들은 강제로 이 상수 인터페이스를 구현해야하며, 이는 **이름 공간의 오염**을 초래한다.

> java.io.ObjectStreamConstants 등의 상수 인터페이스는 인터페이스를 잘못 사용한 예다.
>

- 상수를 공개할 목적이라면
    - 특정 클래스나 인터페이스와 강하게 연관된 상수라면 그 클래스나 인터페이스 자체에 추가해야 한다.

  ex) Integer와 Double에 선언된 MIN_VALUE, MAX_VALUE

    - 그것도 아니라면 **인스턴스화 할 수 없는 유틸리티 클래스**에 담아 공개하라.

    ```jsx
    package effectivejava■chapter4.item22.constantutilityclass;
    public class PhysicalConstants {
    private PhysicalConstants（） { } // 인스턴스화 방지
    // 아보가드로 수 （1/몰）
    public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    // 볼츠만 상수 （J/K)
    public static final double BOLTZMANN_CONST = 1.380_648_52e-23;
    // 전자 질량 （kg）
    public static final double ELECTRON_MASS = 9.109_383_56e-31;
    }
    ```

  > 고정 소수점, 부동 소수점, 십진수 리터럴 등등 뭐든 밑줄을 이용하는 것을 고려하라
  >

    - 유틸리티 클래스에 정의된 상수를 사용하려면 클래스 이름까지 명시해야한다.

      ex) PhysicalConstants.AVOGADROS_NUMBER

    - 이를 자주 쓴다면 정적 임포트하여 클래스 이름 생략도 가능하다.

        ```jsx
        import static effectivejava.chapter4.item22.constantutilityclass.PhysicalConstants. * ;
        public class Test {
        double atoms(double mots) {
        return AVOGADROS_NUMBER * mols;
        }
        // PhysicalConstants를 빈번히 사용한다면 정적 임포트가 값어치를 한다.
        }
        ```


📁 정리 **: 인터페이스는 타입을 정의하는 용도로만 쓰고, 상수 공개용 수단으로 쓰지 마라.**