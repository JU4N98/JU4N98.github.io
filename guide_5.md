# Programacion Competitiva I

#### Armar los equipos

**Problema:** dado 10 jugadores, cada uno con rating $r_{i}$, armar 2 equipos de forma tal que la diferencia de rating entre ambos sea minima.

**Solucion:** armar uno de los equipos y luego calcular el rating del otro con una resta.

**Complejidad:** para armar un equipo se deben armar todas las combinaciones de 5 jugadores tomando de un grupo de a 10, aproximadamente se realizan $(10^5)$ operaciones. Si el tamano del equipo es N y la cantidad de jugadores es K, entonces la complejidad es $O(N^K)$.

**Codigo:**
```cpp
vector<int> score(10); 
int sum = 0;
forn(i,10){
	cin >> score[i];
	sum += score[i];
}

int ans = INT_MAX;
forr(p1,0,10){
	forr(p2,p1+1,10){
		forr(p3,p2+1,10){
			forr(p4,p3+1,10){
				forr(p5,p4+1,10){
					int cur = score[p1] + score[p2] + score[p3] + score[p4] + score[p5];
					ans = min(ans,abs(sum-2*cur));
				}
			}
		}
	}
}
```

#### Indumentaria

**Problema:** dado N tipos de prendas, cada una con un precio asociado $price_{i}$ y M relaciones entre prendas (cada relacion esta formada por dos enteros A y B que indican que la prenda A matchea con la prenda B), determinar cual es el minimo precio a pagar por tres ropas que matcheen entre si.

**Solucion:** por cada relacion de matcheo A-B (A matchea con B), probar si existe una ropa C, diferente a A y B, tal que las relaciones A-C y B-C existen e ir llevando el precio minimo.

**Complejidad:** la complejidad del algoritmo es $O(M * N)$. 

**Codigo:**
```cpp
int n, m; cin >> n >> m;
vector<int> price(n); forn(i,n) cin >> price[i];
vector<vector<bool>> match(n,vector<bool>(n));
vector<ii> pairs;
forn(i,m){
	ii in; cin >> in.fst >> in.snd;
	in.fst--, in.snd--;
	match[in.fst][in.snd] = match[in.snd][in.fst] = true;
	pairs.pb(in);
}

int ans = INT_MAX;
forn(i,m) {
	ii cur = pairs[i];
	forn(j,n) {
		if(j == cur.fst || j == cur.snd) continue;
		if(match[cur.fst][j] && match[cur.snd][j]){
			ans = min(ans, price[cur.fst] + price[cur.snd] + price[j]);
		}
	}
}

cout << (ans == INT_MAX ? -1 : ans) << "\n";
```

#### Tres Hijos

**Problema:** dada una matriz con N filas y M columnas, cada celda tiene asociado una cantidad de granos que produce $C_{ij}$, calcular la forma de dividir el campo en 3 partes de forma que cada una produzca una cantidad esperada. Para dividir el campo se hacen dos cortes verticales o dos horizontales y cada parte debe ser no vacia.

**Solucion:** calcular la suma de granos para cada columna, luego sobre este arreglo calcular el prefix sum del mismo, finalmente probar todos los cortes posibles y calcular la cantidad de granos de cada corte. Si el corte es valido se incrementa en 1a respuesta. Repetir este procedimiento para las filas.

**Complejidad:** la complejidad del algoritmo es $O(M*M + N*N)$, ya que tengo que probar todas las combinaciones de 2 cortes para filas y columnas, y la verificacion de si es un corte valido o no se puede hacer en O(1) gracias al prefix sum.

**Codigo:**
```cpp
int solve (const vector<int> &corn, const vector<int> &expected) {
	int n = sz(corn);
	vector<int> prefix(n+1);
	forn(i,n) prefix[i+1] = prefix[i] + corn[i];
	
	int ans = 0;
	vector<int> cur(3);
	forr(i,0,n) {
		forr(j,i+1,n-1) {
			cur[0] = prefix[i+1];
			cur[1] = prefix[j+1]-prefix[i+1];
			cur[2] = prefix[n] - prefix[j+1];
			sort(cur.begin(),cur.end());
			ans += (cur[0] == expected[0] && cur[1] == expected[1] && cur[2] == expected[2]);
		}
	}
	
	return ans;
}

int n,m; cin >> n >> m;
vector<int> rows(n), columns(m), expected(3);
forn(i,n) {
	forn(j, m) {
		int corn; cin >> corn;
		rows[i] += corn;
		columns[j] += corn;
	}
}

forn(i,3) cin >> expected[i];
sort(expected.begin(), expected.end());

int ans = solve(rows, expected);
ans += solve(columns, expected);

cout << ans << "\n";

```