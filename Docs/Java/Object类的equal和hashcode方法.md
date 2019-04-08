### hashcode（）和equals（）的作用、区别、联系
因为hashCode()并不是完全可靠，有时候不同的对象他们生成的hashcode也会一样（生成hash值得公式可能存在的问题），所以hashCode()只能说是大部分时候可靠，并不是绝对可靠，所以我们可以得出：
1.equal()相等的两个对象他们的hashCode()肯定相等，也就是用equal()对比是绝对可靠的。
2.hashCode()相等的两个对象他们的equal()不一定相等，也就是hashCode()不是绝对可靠的。