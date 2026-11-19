CarroRepository
package com.cotemig.prova.data

import kotlinx.coroutines.delay

object CarroRepository {

    private val itens = listOf(
        Carro(
            1,
            "Fusca",
            "Volkswagen",
            1970,
            "Ícone brasileiro, motor traseiro refrigerado a ar."
        ),
        Carro(
            2,
            "Gol",
            "Volkswagen",
            1995,
            "Compacto mais vendido do país por décadas."
        ),
        Carro(
            3,
            "Uno",
            "Fiat",
            1990,
            "Hatch pequeno e econômico."
        ),
        Carro(
            4,
            "Corolla",
            "Toyota",
            2020,
            "Sedã médio conhecido pela confiabilidade."
        ),
        Carro(
            5,
            "Onix",
            "Chevrolet",
            2018,
            "Hatch moderno e campeão de vendas."
        )
    )

    suspend fun listar(): List<Carro> {
        delay(300)
        return itens
    }

    suspend fun buscarPorId(id: Int): Carro? {
        delay(800)
        return itens.find { it.id == id }
    }
}
2-CarroViewModel.kt
package com.cotemig.prova.viewmodel

import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.setValue
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope

import com.cotemig.prova.data.Carro
import com.cotemig.prova.data.CarroRepository

import kotlinx.coroutines.launch
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.update

class CarroViewModel : ViewModel() {

    var carros by mutableStateOf<List<Carro>>(emptyList())
        private set

    init {
        viewModelScope.launch {
            carros = CarroRepository.listar()
        }
    }

    private val _favoritoIds =
        MutableStateFlow<Set<Int>>(emptySet())

    val favoritoIds: StateFlow<Set<Int>> =
        _favoritoIds.asStateFlow()

    suspend fun buscarCarro(id: Int): Carro? {
        return CarroRepository.buscarPorId(id)
    }

    fun alternarFavorito(id: Int) {
        _favoritoIds.update { favoritos ->
            if (id in favoritos) {
                favoritos - id
            } else {
                favoritos + id
            }
        }
    }
}
ListaCarroScreen.kt
package com.cotemig.prova.userinterface

import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material3.Card
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp

import com.cotemig.prova.viewmodel.CarroViewModel

@Composable
fun ListaCarroScreen(
    viewModel: CarroViewModel,
    aoClicar: (Int) -> Unit
) {

    val favoritos by viewModel.favoritoIds.collectAsState()

    LazyColumn(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),

        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {

        items(viewModel.carros) { item ->

            Card(
                modifier = Modifier
                    .fillMaxWidth()
                    .clickable {
                        aoClicar(item.id)
                    }
            ) {

                Row(
                    modifier = Modifier.padding(16.dp),
                    verticalAlignment = Alignment.CenterVertically
                ) {

                    Column(
                        modifier = Modifier.weight(1f)
                    ) {

                        Text(
                            text = item.modelo,
                            style = MaterialTheme.typography.titleMedium
                        )

                        Text(
                            text = item.marca,
                            style = MaterialTheme.typography.bodyMedium
                        )
                    }

                    if (item.id in favoritos) {
                        Text("★")
                    }
                }
            }
        }
    }
}
DetalheCarroScreen.kt
package com.cotemig.prova.userinterface

import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.wrapContentSize
import androidx.compose.material3.Button
import androidx.compose.material3.CircularProgressIndicator
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp

import com.cotemig.prova.data.Carro
import com.cotemig.prova.viewmodel.CarroViewModel

@Composable
fun DetalheCarroScreen(
    viewModel: CarroViewModel,
    carroId: Int,
    aoVoltar: () -> Unit
) {

    var carro by remember(carroId) {
        mutableStateOf<Carro?>(null)
    }

    val favoritos by viewModel.favoritoIds.collectAsState()

    LaunchedEffect(carroId) {
        carro = viewModel.buscarCarro(carroId)
    }

    if (carro == null) {

        Column(
            modifier = Modifier
                .fillMaxSize()
                .wrapContentSize(Alignment.Center)
        ) {

            CircularProgressIndicator()
        }

    } else {

        val item = carro!!

        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(24.dp),

            verticalArrangement = Arrangement.Center,
            horizontalAlignment = Alignment.CenterHorizontally
        ) {

            Text(
                text = item.modelo,
                style = MaterialTheme.typography.headlineLarge
            )

            Spacer(modifier = Modifier.height(16.dp))

            Text(
                text = "Marca: ${item.marca}"
            )

            Spacer(modifier = Modifier.height(8.dp))

            Text(
                text = "Ano: ${item.ano}"
            )

            Spacer(modifier = Modifier.height(8.dp))

            Text(
                text = "Descrição: ${item.descricao}"
            )

            Spacer(modifier = Modifier.height(24.dp))

            Button(
                onClick = {
                    viewModel.alternarFavorito(item.id)
                }
            ) {

                Text(
                    text = if (item.id in favoritos) {
                        " ★ Favorito"
                    } else {
                        " ☆ Favoritar"
                    }
                )
            }

            Spacer(modifier = Modifier.height(16.dp))

            Button(
                onClick = aoVoltar
            ) {

                Text("← Voltar")
            }
        }
    }
}
AppNavigation.kt
package com.cotemig.prova.userinterface

import androidx.compose.runtime.Composable
import androidx.lifecycle.viewmodel.compose.viewModel
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.rememberNavController
import androidx.navigation.toRoute

import com.cotemig.prova.viewmodel.CarroViewModel

import kotlinx.serialization.Serializable

@Serializable
sealed interface Rota {

    @Serializable
    data object Lista : Rota

    @Serializable
    data class Detalhe(val carroId: Int) : Rota
}

@Composable
fun AppNavigation() {

    val navController = rememberNavController()

    val viewModel: CarroViewModel = viewModel()

    NavHost(
        navController = navController,
        startDestination = Rota.Lista
    ) {

        composable<Rota.Lista> {

            ListaCarroScreen(
                viewModel = viewModel,

                aoClicar = { id ->

                    navController.navigate(
                        Rota.Detalhe(id)
                    )
                }
            )
        }

        composable<Rota.Detalhe> { backStackEntry ->

            val rota = backStackEntry.toRoute<Rota.Detalhe>()

            DetalheCarroScreen(
                viewModel = viewModel,

                carroId = rota.carroId,

                aoVoltar = {
                    navController.popBackStack()
                }
            )
        }
    }
}
MainActivity.kt
package com.cotemig.prova

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.safeDrawingPadding
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.ui.Modifier

import com.cotemig.prova.userinterface.AppNavigation

class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {

            MaterialTheme {

                Surface(
                    Modifier
                        .fillMaxSize()
                        .safeDrawingPadding()
                ) {

                    AppNavigation()
                }
            }
        }
    }
}
