<h1 style="text-align:center">Prova in Itinere del 8 Febbraio 2007</h1>

### 1) Si consideri la seguente variante di El Gamal, che potremmo chiamare El Gamal'. Il cifrario El Gamal' ha come unica differenza, rispetto a El Gamal, che $M = \{0,...,p - 1\}$. È El Gamal' un cifrario sicuro?  Giustificare formalmente la risposta fornita

No, il cifrario $\mathcal{ElGamal}'$ non è sicuro. Se $\mathcal{M} = \{0, 1, ..., p-1\}$, posso banalmente selezionare un messaggio $m = 0, m \in \mathcal{M}$. In $ESP_{\mathcal{ElGamal'}}^{ind-cpa}(A)$, basta che l'avversario chieda la cifratura di un messaggio casuale e $m = 0$. Basta controllare che l'output dell'oracolo è uguale a 0 o no per determinare quale dei due messaggi è stato cifrato.

Più genericamente, il problema è che permettendo di cifrare messaggi $m \notin G$, nell'operazione $C_2 = h^rm \mod p$ c'è la possibilità che  $C_2 \notin G$, fatto che potrebbe rendere possibile distinguere una cifratura di un messaggio rispetto ad un'altra.

### 2)  Descrivere dettagliatamente il cifrario Paillier (nella versione semplificata presentata in classe). In particolare, descrivere l’algoritmo di generazione della chiave, l’algoritmo di cifratura e l’algoritmo di decifratura.

Il cifrario di Paillier è un cifrario asimmetrico che sfrutta il la difficoltà del problema decisionale della n-residuosità.

È composto dai seguenti algoritmi:

```pseudocode
KEY-GEN(): /* sia k il numero di bit che N deve avere */
	l_1 <- floor(k/2)
	l_2 <- cail(k/2)
	N <- 0
	while N == 0:
        p <-R- {0, 1}^{l_1}
        q <-R- {0, 1}^{l_2}
      	if TEST-PRIME(p) and TEST-PRIME(q) and p != q and 2**(k - 1) <= N:
      		N <- p*q
    return (N, (p, q)) /* N è la chiave pubblica, (p, q) la chiave privata */
  
ENC(N, m):
	y <-R- Z_n
  	c <- (1 + mN)y**N % N**2 /* (1 + mN) e y**N sono entrambi quadrati residui in N**2 */
  	return c
  
DEC((p, q), c):
	phi <- (p - 1) * (q - 1)
	c1 <- c ** phi /* elimina y**N e lascia (1 + mN)**phi */
	d <- FIND-INVERSE(phi, N**2)
	c2 <- c1 ** d /* elimina **phi e lascia (1 + mN) */
	m <- (c2 - 1) / N
	return m
```

### 3) Definire formalmente il concetto di sicurezza (ovvero non falsificabilità relativamente ad attacchi a messaggio scelto) per schemi di firma digitale.

Sia $\mathcal{DS} = (\mathcal{KeyGen}, \mathcal{Sign}, \mathcal{VF})$ uno schema di firma digitale. 

Si consideri quindi il seguente esperimento: ad un avversario $A$ viene data in input la chiave di verifica e viene fornito un oracolo di firma $O_{\mathcal{Sign}}$ che prende in input un messaggio e restituisce la corrispettiva firma (o tag). L'obiettivo di $A$ è restituire in ouput una coppia (messaggio, tag) che risulti valida per la funzione di verifica $\mathcal{VF}$. Il messaggio nella coppia non può essere stato dato precedentemente in input all'oracolo. Si ricordi che $A$ può autonomamente verificare la validità di un messaggio utilizzando la funzione $\mathcal{VF}$.

```pseudocode
ESP_{DS}^{uf-cma}(A):
	sk, vk <- KeyGen()
	(m, tag) <- A(vk)
	if VF(m, tag) != 1 or A cheats:
		return 0
	return 1
```

Il vantaggio di $A$ in senso **uf-cma** è così definito
$$
Adv_{\mathcal{DF}}^{uf-cma}(A) = Pr[ESP_{\mathcal{DF}}^{uf-cma}(A) = 1]
$$
Lo schema di firma digitale $\mathcal{DS} = (\mathcal{KeyGen}, \mathcal{Sign}, \mathcal{VF})$ è sicuro in senso **uf-cma** se il vantaggio è prossimo a 0 per ogni avversario polinomialmente limitato.

### 4) Questo schema è sicuro? Giustificare formalmente la risposta fornita. (VEDERE COMPITO)

No, lo schema di firma digitale proposto non è sicuro. Si facciano le seguenti considerazioni:
$$
m_1 \xleftarrow{$}\mathcal{M}, \quad \sigma_1 = \root{m_1}\of{S} = S^{\frac{1}{m_1}} \\
m_2 \xleftarrow{$}\mathcal{M}, \quad \sigma_2 = \root{m_2}\of{S} = S^{\frac{1}{m_2}} \\
\\
\text{Notando che: } \\
\sigma_1 \cdot \sigma_2 = S^{\frac{1}{m_1}} \cdot S^{\frac{1}{m_2}} = S^{\frac{1}{m_1 \cdot m_2}} \\
\text{Possiamo definire} \\
m_3 = m_1 \cdot m_2, \quad \sigma_3 = \sigma_1 \cdot \sigma_2
$$

```pseudocode
A(VK):
	m1 <-R- M
	m2 <-R- M
	s1 <- O_{Sign}(m1)
	s2 <- O_{Sign}(m2)
	m3 <- m1 * m2
	s3 <- s1 * s2
	return (m3, s3)
```

Questo avversario ha vantaggio pari a 1 per quanto detto in precedenza.

