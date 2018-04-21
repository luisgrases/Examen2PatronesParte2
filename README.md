# Examen 2 Parte 2
## Template Pattern
Para implementar el juego de mesa requerimos que las fichas puedan tener distintos tipos de movimiento. Cada vez que se requiere hacer un movimiento se verifica si el movimiento es válido. Si el movimiento es válido se procede a mover la ficha. Se puede observar como dentro de esta funcionalidad el tipo de movimiento puede cambiar; por lo tanto el código que verifica si el movimiento es valido cambia según el movimiento que se este haciendo. Por otro lado una vez que esta verificación es terminada, el código a ejecutar es constante: Se mueve la ficha o no se mueve. Para esta situación se utiliza el template pattern, como se puede ver en el código siguiente:
```
public abstract class Piece {
	private Board board;
	private int x, y;

	protected abstract boolean isValidMove(int x, int y);

	public void move(int x, int y) {
		if(isValidMove(x, y)) {
			setPosition(x, y);
		}
	}
}
```
A partir de esta clase abstracta podemos implementar subclases que representarían las fichas especificas. 

```
public class Rey extends Piece {

	@Override
	protected boolean isValidMove(int x, int y) {
		//...
	}

} 
```
Una vez implementado `isValidMove` para dichas subclases, se puede utilizar el método `move`.

Gracias al template pattern podemos separar el codigo que cambia del que no cambia en este caso específico.


## Strategy Pattern 
Algunos juegos de mesa pueden requerir que las fichas cambien su comportamiento durante el juego. Para ello se pueden utilizar comportamientos intercambiables en tiempo de ejecución. El Strategy Pattern permite hacer exactamente eso. En lugar de que cada ficha defina su tipo de movimiento, se pueden definir tipos de movimiento e inyectarlos a las mismas.

Definimos la interfaz para los movimientos:
```
public interface MovementBehaviour {
	boolean isValidMove(int x1, int y1, int x2, int y2);
}
```
Ejemplo de movimiento:
```
public class DiagonalMovementBehaviour implements MovementBehaviour {

	@Override
	public boolean isValidMove(int x1, int y1, int x2, int y2) {
		//...
	}
	
}
```
 Ahora las fichas tienen un nuevo atributo llamado movementBehaviour, ya no implementan movimiento si no que tienen una referencia a el:
```
public abstract class Piece {

	public Board board;
	public int x, y;
	private MovementBehaviour movementBehaviour;
	
public void move(int x, int y) {
	if(movementBehaviour.isValidMove(this.x, this.y, x, y)) {
		setPosition(x, y);
	}
} 

public setMovementBehaviour(MovementBehaviour movementBehaviour) {
	//...
}
	```

Utilizando `setMovementBehaviour` se inyecta el tipo de movimiento deseado.



## Observer Pattern 
Una vez que se comprueba que el movimiento es valido tanto para la ficha como para el estado del tablero, se actualizan las coordenadas de la ficha. Necesitamos notificar al tablero de que esta ficha actualizó sus coordenadas para poder actualizar su posición en el tablero. Para ello se utiliza el Observer Pattern.
```
public void move(int x, int y) {
	if(movementBehaviour.isValidMove(this.x, this.y, x, y)) {
		setPosition(x, y);
		notifyBoard();
	}
}
```
```
private void notifyBoard() {
	this.board.update();
}
```
```
public interface Observer {
   public void update();
}
```
```
public class Board extends Observer {
	int[][] tiles = new int[8][8];
	
	 @Override
	   public void update() {
	      //actualizar tiles
	   }
}
```
## Factory Pattern
A la hora de cargar las partidas, el proceso varia dependiendo de el formato en el cual esta guardada la misma. Necesitamos un deserializador o “cargador” distinto para cada tipo de formato. Como no se sabe que tipo de formato se va a cargar antes de que esto ocurra en runtime, debemos hacer un switch statement que verifique el formato, y en función del mismo, instanciar el objeto "cargador" requerido. El Factory Pattern nos ayuda a hacer esto de una manera limpia. Encapsulando el proceso de instanciación en una clase aparte. Además asegura que todos los objetos instanciados cumplan una interfaz especifica.

```
public class GameLoaderJSON extends GameLoader {
	
	// Constructor…
	
	 @Override
	   public void load(Board board) {
	      // load game
	   }
	
}
```
```
public class GameLoaderXML extends GameLoader {
	
	// Constructor…
	
	 @Override
	   public void load(Board board) {
	      // load game 
	   }
}
```
```
public interface GameLoader {
	void load(Game game);
}
```
```
public class GameLoaderFactory {
	public GameLoader get(GameLoaderFormat format) {
		switch(format) {
			case JSON: return new GameLoaderJSON();
			case XML: return new GameLoaderXML();
			default: return null;
		}
	}
}
```
```
enum GameLoaderFormat {
	JSON,
	XML
} 
```
Luego podemos usarlo como:
```
GameLoaderFactory gameLoaderFactory = new GameLoaderFactory();
gameLoaderFactory.get(tipo).load();
```
