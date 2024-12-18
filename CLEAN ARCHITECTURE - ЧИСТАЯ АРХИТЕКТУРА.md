Кол-во слоев может меняться, как в большую так и в меньшую сторону. Существует 3 правила, следуя которым можно переделать архитектуру, что значит `оставить плюсы` и `убрать сложности`:
- **Правило пересечения границ слоев**:
	- Границы пересекают модели, которые по сути являются паттерном `Data Transfer Object`.
- **Слой бизнес логики не зависит от других слоев**:
	- Здесь `dependency injection (внедрение зависимостей)` только интерфейсы.
- **Приоритет внутреннего слоя**:
	- Слой бизнес логики определяет интерфейс, по которому осуществляется взаимодействие бизнес логики с внешним миром (со сторонней системой, с БД, с third-party API и тд).