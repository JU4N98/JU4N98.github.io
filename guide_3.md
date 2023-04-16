# Programacion Competitiva

#### 1415 - Broken Keyboard

**Problema:** dado un string S, que indica las teclas presionadas en un teclado, obtener el string escrito en pantalla, sabiendo que los caracteres especiales `[` y `]`, mueven el cursor al principio y al final de la pantalla respectivamente.

**Solucion:** usar una deque que tiene inserciones en O(1) en los extremos, si aparece el caracter `[` comenzar a insertar al principio, si aparece `]` comenzar a insertar al final.

**Complejidad**: $O(N)$, siendo $N$ la cantidad de caracteres del string.

**Codigo:**
```cpp
string s;
while(cin >> s){
	deque<char> ans;
	bool add_front = true;
	forn(i,sz(s)){
		int j = i;
		while(j<sz(s) && s[j] != '[' && s[j] != ']') j++;
		
		if(add_front) for(int k = j-1; k>=i; k--) ans.push_front(s[k]);
		else forr(k,i,j) ans.pb(s[k]);
		
		if(j<sz(s) && s[j] == '[') add_front = true;
		if(j<sz(s) && s[j] == ']') add_front = false;
		i = j;
	}
	
	forn(i,sz(ans)) cout << ans[i];
	cout << "\n";
}
```
---
#### 2065 - Supermarket Line
**Problema:** dado una fila de clientes (donde cada cliente lleva una cantidad $C_{i}$ de items) que se atienden en el orden dado, y dado ciertos cajeros (donde cada cajero procesa un item en un tiempo $V_{i}$), determinar el tiempo en el que se atienden a todos los clientes, sabiendo que si hay multiples cajeros disponibles, el de menor indice es el que atiende al siguiente cliente.

**Solucion:** este es un problema de simulacion:
* Llevar la cantidad de items que lleva cada cliente en una queue.
* Llevar los cajeros disponibles en una priority queue ordenados por menor id primero.
* Llevar los eventos cuando se desocupa un cajero en una priority queue, ordenados por menor tiempo.

Mientras haya eventos en la priority queue y clientes en la queue:
1. Actualizar el tiempo actual. 
2. Desocupar todos los cajeros en el tiempo actual.
3. Atender a todos los clientes posibles.

**Complejidad:** $O(N*log(N))$ siendo $N$ la cantidad de clientes.

**Codigo:**
```cpp
int n,m; cin >> n >> m;

priority_queue<int, vector<int>, greater<int>> checker;
vector<ll> v(n);
forn(i,n){
	cin >> v[i];
	checker.push(i);
}

queue<ll> client; 
forn(i,m){
	ll c; cin >> c;
	client.push(c);
}

priority_queue<pair<ll,int>, vector<pair<ll,int>>,greater<pair<ll,int>>> events;

ll t = 0;
while(sz(client) || sz(events)){
	if(sz(events)) t = events.top().fst;
	
	while(sz(events) && t == events.top().fst){
		checker.push(events.top().snd);
		events.pop();
	}
	
	while(sz(client) && sz(checker)){
		int checker_id = checker.top(); checker.pop();
		
		ll c = client.front(); client.pop();
		
		events.push({t + v[checker_id]*c,checker_id});
	}
}

cout << t << "\n";
```
---
#### 1633 - SBC

**Problema:** dada una CPU que puede ejecutar un proceso a la vez, y un conjunto de procesos, donde el i-esimo se encuentra definido por $t_i$ (el tiempo de llegada del proceso a la CPU, no puede ser ejecutado antes de su llegada) y $c_i$ (las unidades de tiempo de CPU que consume), determinar la menor sumatoria de espera del conjunto de procesos.

**Solucion:** este tambien es un problema de simulacion:
* Llevar los eventos en una priority queue, los eventos son dos:
    * Evento 1: llega un proceso a la CPU.
    * Evento 2: se desocupa la CPU.
* Llevar en una priority queue los $C_{i}$ de los procesos listos para ejecutar, con mayor prioridad los que tienen menor $C_{i}$.

Falta definir el orden en el que se procesan los eventos con igual tiempo y que se hace cuando llega cada evento:
* Primero se procesan los eventos de llegada de proceso a CPU.
* Si llega:
    * Evento 1: si la CPU esta disponible creamos un evento de liberacion de CPU con tiempo igual a $t_{actual} + C_{i}$, si no esta disponible la CPU se inserta el $C_i$ en la priority queue de procesos listos para ejecutar.
    * Evento 2: si hay procesos listos para ejecutar, el proceso de menor tiempo de ejecucion pasa a ocupar la CPU y se inserta un nuevo evento de liberacion de CPU con tiempo igual a $t_{actual} + C_{i}$.

Para llevar los tiempos de demora, cuando se ejecuta un proceso se suma a la respuesta la diferencia entre el tiempo actual y el tiempo de llegada a CPU del proceso que se va a ejecutar, es decir, $t_{actual} - t_i$.

**Complejidad:** $O(N * log(N))$, siendo $N$ la cantidad de procesos.

**Codigo:**
```cpp
int n;
typedef vector<ll> event;

int n;
while(cin>>n){
	priority_queue<event, vector<event>, greater<event>> events;
	priority_queue<pair<ll,ll>, vector<pair<ll,ll>>, greater<pair<ll,ll>>> ready;
	
	forn(i,n){
		ll t,c; cin >> t >> c;
		events.push({t,c,0});
	}
	
	ll ans = 0;
	bool used = false;
	while(sz(events)){
		 event ev = events.top();
		 events.pop();
		 
		 if(ev[2] == 0){
			 if(used){
				 ready.push({ev[1],ev[0]});
			 }else{
				 events.push({ev[0]+ev[1],1010,1});
				 used = true;
			 }
		 }else{
			 if(sz(ready)){
				ans += ev[0] - ready.top().snd;
				events.push({ready.top().fst + ev[0],1010,1});
				ready.pop();
			 }else used = false;
		 }
	}
	cout << ans << "\n";
}	
```