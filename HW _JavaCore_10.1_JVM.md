## Исходный код:

public class JvmComprehension {

    public static void main(String[] args) {
        int i = 1;                      // 1
        Object o = new Object();        // 2
        Integer ii = 2;                 // 3
        printAll(o, i, ii);             // 4
        System.out.println("finished"); // 7
    }

    private static void printAll(Object o, int i, Integer ii) {
        Integer uselessVar = 700;                   // 5
        System.out.println(o.toString() + i + ii);  // 6
    }
}

## Что происходит:

### Шаг 1.
ClassLoader загружает в метаспейс класс JvmComprehension, начинается выполнение метода **main** (т.к. загрузка классов "ленивая", то классы начинают загружатся по мере упоминания в коде).\
В стеке в фрейме для метода **main** создается переменная типа _int_, которой присваивается значение, расположенное в правой части выражения, в данном случае 1.

### Шаг 2.
ClassLoader загружает в метаспейс класс Object.\
Благодаря слову _new_ в куче (heap) выделяется место для создания нового объекта типа Object (тип указывает, сколько памяти понадобится для объекта), конструктор создает этот объект. Переменной _о_, которая расположена в текущем фрейме метода **main**, присваивается этот объект, точнее ссылка на него.

### Шаг 3.
ClassLoader загружает в метаспейс класс Integer.\
Тип Integer является ссылочным типом данных и хранится в куче. Для него выделяется место, размер которого соответствует данному типу, туда помещается значение 2. Переменная _ii_ расположена в стеке, фрейм метода **main**, этой переменной присваивается ссылка на объект.

### Шаг 4.
В данной строке вызывается новый метод **printAll**, для которого в стеке создается новый фрейм и теперь этот новый фрейм становится текущим. Переменные _o, i_ и _ii_ будут переданы в текущий фрейм параметрами.\
Для выполнения метода потребуются классы Object и Integer, но они уже были загружены ранее.

### Шаг 5.
В куче выделится место для создания нового объекта _Integer_ со значением 700. В текущем фрейме метода **printAll** создастся новая переменная _uselessVar_, которая будет ссылаться на этот объект, расположенный в хипе.\
Класс Integer будет взят из кэша.

### Шаг 6.
В этой строке снова вызывается новый метод **System.out.println**, для которого в стеке создается еще один фрейм (теперь их 3).\
Данный метод, в свою очередь, вызывает еще один метод **toString()**, для которого тоже создается фрейм (теперь в стеке уже 4 фрейма). В результате вызова **toString()** в куче будет создан новый объект типа String, в конструктор которого будет передано значение объекта _Object o_.
ClassLoader загружает в метаспейс класс String. После создания объекта метод завершаеся, его фрейм удаляется.\
Результат выполнения метода **_toString()_** передается (возвращается) в текущий фрейм метода **System.out.println**. Локальные переменные _i_ и _ii_ уже есть в этом фрейме (они были переданы параметрами в шаге 4).\
Выполнение метода **System.out.println** завершается, его фрейм удаляется, а объект типа String будет удален сборщиком мусора, т.к. на него больше нет ссылок. Следующим за ним в стеке (верхний в стопке) идет фрейм метода **printAll**.\
Т.к. **System.out.println** завершился, метод **printAll** теперь тоже может завершиться, его фрейм удаляется вместе с переменной _uselessVar_, следовательно, объект _Integer(700)_, на который она ссылалась, удаляется из кучи сборщиком мусора. В стеке остался только один фрейм метода **main**, который становится текущим.

### Шаг 7.
Продолжается выполнение метода **main**. На этом шаге основной метод вызывает метод **System.out.println**, для которого создается фрейм (теперь в стеке их становится 2).\
В данном случае фрейм метода **System.out.println** создает в куче новый объект типа String со значением _"finished"_ и хранит в себе ссылку на него. 
Для выполнения метода потребуется класс String, но он уже был загружен ранее.
Выполнение метода завершается, текущий фрейм метода **System.out.println** удаляется, ссылка на объект _String_ со значением _(finished)_ больше не хранится, значит объект будет удален сборщиком мусора.\
После удаления фрейма метода **System.out.println** первым (т.е. текущим) в стеке становится фрейм метода **main**, но он теперь тоже завершается. Фрейм удаляется, стек пуст.

