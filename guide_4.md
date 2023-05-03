# Programacion Competitiva I

#### 1431 - Klingon Levels

**Problema:** dada N divisiones, con $K_i$ alumnos en la i-esima division, donde cada alumno tiene asociado un nivel $T_i$, elegir un T que divida cada division en nivel basico (los alumnos con $T_i <= T$) y nivel avanzado (los alumnos con $T_i > T$) que minimice el acumulado de diferencias de cada tipo. 

**Solucion:** calcular la respuesta para cada T posible, para esto: iteramos sobre cada division, sobre cada T posible y sumamos la diferencia obtenida para dicho T en dicha division.

**Complejidad:** por cada division, itero sobre cada T posible, y busco el primer alumno que sea mayor a T, si tenemos los estudiantes de cada division ordenados podemos hallar con la funcion _upper_bound_ dicho elemento en $O(log(K))$. La complejidad resultante es $O(N*T*log(MAXK))$

**Codigo:**
```cpp
int n; cin >> n;
	while(n != 0){
		vector<ll> marks(1001);
		
		forn(N,n){
			ll k; cin >> k;
			vector<ll> division(k);
			forn(i,k) cin >> division[i];
			sort(division.begin(), division.end());
			
			forn(mark,1001){
				ll basic = (upper_bound(division.begin(), division.end(), mark) - division.begin());
				marks[mark] += abs(basic - (k-basic));
			}
		}
		
		ll mini = 0;
		forn(i,1001) if(marks[mini] > marks[i]) mini = i;
		cout << marks[mini] << "\n";
		cin >> n;
	}
```

---
#### 1023 - Drought
**Problema:** dado N propiedades, definidas por un consumo Y y una cantidad de habitantes X:
* Imprimir los consumo por habitantes ordenados ascendentemente junto con la cantidad de personas con dicho consumo.
* Imprimir el promedio del consumo de todos los habitantes

**Solucion:** 
* Por cada propiedad agregar X a la llave del mapa que tenga el valor Y/X, luego imprimimos estos pares llave-valor en forma ascendente.
* Tambien llevamos el acumulado de consumo y la cantidad de habitantes, para poder imprimir el promedio

**Complejidad:** cada insercion en el mapa es $O(log(N))$, como se hace por cada propiedad, la complejidad resultante es $O(N*log(N))$.

**Codigo:**
```cpp
int n; cin >> n;
	int cit = 1;
	while(n != 0){
		if(cit != 1) cout << "\n";
		int sum = 0, cnt = 0;
		map<int,int> ans;
		forn(i,n){
			int x,y; cin >> x >> y;
			ans[y/x] = ans[y/x] + x;
			sum += y;
			cnt += x;
		}
		
		cout << "Cidade# " << cit << ":\n";
		forall(it,ans) cout << it->snd << "-" << it->fst << " ";
		cout << "\n";
		cout << fixed << setprecision(2) << "Consumo medio: " << floor(double(sum)*100/double(cnt))/100 << " m3.\n";
		cit++;
		cin >> n;
	}
```
---
#### 2012 - Height Map

**Problema:** dado una matriz de enteros, donde cada entero representa la altura de la celda, calcular la cantidad de lados que posee la figura.

**Solucion:** la solucion tiene varias partes:
* Podemos observar que inicialmente tenemos las paredes de los costados (4), mas la de la base (1).
* Paredes que forman el techo: todas las celdas que tengan la misma altura y esten conectadas, suman una pared. Esto se puede resolver con el algoritmo _floodfill_:
* Paredes laterales: para las paredes laterales tenemos que definir cuando existe una pared y como saber si las paredes estan conectadas:
    * Existe una pared cuando dos celdas adyacentes tienen diferente altura.
    * Una pared continua si la diferencia de altura tiene el mismo signo y si comparten al menos una celda de altura.

    Deberiamos hacer este chequeo por columnas y filas.

**Complejidad:**
Cada celda sera recorrida una vez por floodfill $O(R*C)$ y dos veces para resolver los lados $O(R*C)$, la complejidad resultante es $O(R*C)$.

**Codigo:**
Para resolver el techo:
```cpp
const int dr[4] = {1,-1,0,0};
const int dc[4] = {0,0,1,-1};

void floodfill(int r, int c, int R, int C, vector<vector<bool>> &visi, const vector<vector<int>> &mat){
	forn(i,4){
		int nr = R + dr[i], nc = C + dc[i];
		if(nr > -1 && nr < r && nc > -1 && nc < c && !visi[nr][nc] && mat[R][C] == mat[nr][nc]){
			visi[nr][nc] = true;
			floodfill(r, c, nr, nc, visi, mat);
		}
	}
}

int solve_ceil(const vector<vector<int>> &mat){
	int ans = 0;
	
	int r = sz(mat), c = sz(mat[0]);
	vector<vector<bool>> visi(r, vector<bool>(c));
	forn(R,r) forn(C,c) if(!visi[R][C]){
		visi[R][C] = true;
		floodfill(r,c,R,C,visi,mat);
		ans++;
	}
	
	return ans;
}
```

Para resolver los lados:

```cpp

int sgn(int a, int b){
	if (a-b > 0) return 1;
	if (a-b == 0) return 0;
	return -1;
}

bool are_connected(int a1, int b1, int a2, int b2){
	bool connected = sgn(a1,b1) == sgn(a2,b2);
	if(a1 > b1) swap(a1,b1);
	if(a2 > b2) swap(a2,b2);
	ii p1 = {a1,b1};
	ii p2 = {a2,b2};
	if(p1>p2) swap(p1,p2);
	connected &= max(p1.fst,p2.fst) < min(p1.snd,p2.snd);
	return connected;
}

int solve_wall(vector<vector<int>> &mat){
	int ans = 0;
	
	int r = sz(mat), c = sz(mat[0]);
	forr(R,1,r){
		forn(C,c){
			if(mat[R][C] - mat[R-1][C] == 0) continue;
			
			
			int nc = C+1;
			while(nc < c && are_connected(mat[R][nc],mat[R-1][nc], mat[R][nc-1], mat[R-1][nc-1])) nc++;
		
			C = nc-1;
			ans++;
		}
	}
	return ans;
}
```

### 1053 - Dynamic Frog

**Problema:** dado una laguna de largo D, y N rocas, donde cada roca puede ser Big (no se hunde al saltar sobre ella) o Small (se hunde al saltar sobre ella). Minimizar el salto maximo, si la rana tiene que ir y volver saltando por dichas piedras.

**Solucion:** 
* Es facil de ver que siempre tenemos que pisar las piedras Big.
* Ahora, que hacemos con las Small? La solucion es intercalar saltos en las piedras Small a la ida y pisar todas las que no se hayan hundido a la vuelta.

**Complejdad:** hay varias formas de implementarlo, si usamos un map para llevar las piedras y luego borrarlas, podemos hacerlo en complejidad $O(N*log(N))$.

**Codigo:**

```cpp

pair<int,char> parse (const string &in){
	pair<int,char> ret = {0,'S'};
	ret.snd = in[0];
	
	forr(i,2,sz(in))
		ret.fst = ret.fst * 10 + (in[i] - '0');
	
	return ret;
}

int jump (map<int,char> &rock, bool force_jump) {
	
	int pre = 0, ans = 0;
	bool odd = true;
	vector<int> to_erase;
	foraint(it,rock){
		if(it->snd == 'B'){
			ans = max(ans, it->fst - pre); 
			pre = it->fst;
		}else{
			if(odd  || force_jump){
				to_erase.pb(it->fst);
				ans = max(ans, it->fst - pre);
				pre = it->fst;
			}
			odd = !odd;
		}
	}
	
	forn(i,sz(to_erase))
		rock.erase(rock.find(to_erase[i]));
	
	return ans;
}

int t; cin >> t;
forn(T,t){
	int n, d; cin >> n >> d;
	
	map<int,char> rock;
	rock[0] = 'B';
	rock[d] = 'B';
	forn(N,n){
		string in; cin >> in;
		pair<int,char> ret = parse(in);
		assert(rock.count(ret.fst) == 0);
		rock[ret.fst] = ret.snd; 
	}
	
	int ans = jump(rock,false);
	ans = max(ans, jump(rock,true));
	
	cout << "Case " << T+1 << ": " << ans << "\n";
}
```