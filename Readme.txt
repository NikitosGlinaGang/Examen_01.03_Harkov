Харьков Никита
Вариант 21:
«Лайк»
Создайте карточку с текстом «Пост №1» и кнопкой-сердечком (IconButton с иконкой сердца). При нажатии иконка меняется с пустого сердца на заполненное (и обратно). Рядом с иконкой — счётчик лайков. Добавьте анимацию увеличения счётчика при лайке (animateIntAsState или простое изменение размера).
MainActivity.kt
```
package com.example.examen_0103_harkov  // ← ваш package

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.ui.Modifier

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MaterialTheme {  // ← вместо LikeAppTheme
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    LikeScreen()
                }
            }
        }
    }
}
```
LikeCard.kt
```
package com.example.examen_0103_harkov

import androidx.compose.animation.animateColorAsState
import androidx.compose.animation.core.animateFloatAsState
import androidx.compose.animation.core.animateIntAsState
import androidx.compose.animation.core.spring
import androidx.compose.animation.core.tween
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Favorite
import androidx.compose.material.icons.outlined.FavoriteBorder
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.scale
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp

// ─── Экран ───────────────────────────────────────────────────────────────────

@Composable
fun LikeScreen() {
    Box(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        contentAlignment = Alignment.Center/
    ) {
        PostCard(postNumber = 1)
    }
}

// ─── Карточка поста ──────────────────────────────────────────────────────────

@Composable
fun PostCard(postNumber: Int) {
    // Состояние: лайкнуто или нет
    var isLiked by remember { mutableStateOf(false) }

    // Счётчик лайков (анимируется через animateIntAsState)
    var likeCount by remember { mutableStateOf(42) }
    val animatedCount by animateIntAsState(
        targetValue = likeCount,
        animationSpec = tween(durationMillis = 300),
        label = "likeCountAnim"
    )

    // Цвет иконки (анимируется)
    val iconColor by animateColorAsState(
        targetValue = if (isLiked) Color(0xFFE91E63) else Color(0xFF9E9E9E),
        animationSpec = tween(durationMillis = 250),
        label = "iconColorAnim"
    )

    // Масштаб иконки — пружинный эффект при нажатии
    var pressed by remember { mutableStateOf(false) }
    val iconScale by animateFloatAsState(
        targetValue = if (pressed) 1.4f else 1f,
        animationSpec = spring(dampingRatio = 0.3f, stiffness = 400f),
        label = "iconScaleAnim",
        finishedListener = { pressed = false }
    )

    Card(
        modifier = Modifier
            .fillMaxWidth()
            .wrapContentHeight(),
        shape = RoundedCornerShape(16.dp),
        colors = CardDefaults.cardColors(
            containerColor = MaterialTheme.colorScheme.surface
        ),
        elevation = CardDefaults.cardElevation(defaultElevation = 4.dp)
    ) {
        Column(
            modifier = Modifier
                .fillMaxWidth()
                .padding(20.dp)
        ) {
            // Заголовок поста
            Text(
                text = "Пост №$postNumber",
                style = MaterialTheme.typography.titleLarge.copy(
                    fontWeight = FontWeight.Bold,
                    fontSize = 20.sp
                ),
                color = MaterialTheme.colorScheme.onSurface
            )

            Spacer(modifier = Modifier.height(12.dp))

            // Текст поста
            Text(
                text = "Это содержимое поста. Здесь может быть любой текст — " +
                        "интересная история, новость или заметка.",
                style = MaterialTheme.typography.bodyMedium,
                color = MaterialTheme.colorScheme.onSurfaceVariant,
                lineHeight = 22.sp
            )

            Spacer(modifier = Modifier.height(16.dp))

            HorizontalDivider(color = MaterialTheme.colorScheme.outlineVariant)

            Spacer(modifier = Modifier.height(8.dp))

            // Строка с лайком
            Row(
                verticalAlignment = Alignment.CenterVertically,
                horizontalArrangement = Arrangement.Start
            ) {
                // Кнопка-сердечко
                IconButton(
                    onClick = {
                        isLiked = !isLiked
                        likeCount += if (isLiked) 1 else -1
                        pressed = true
                    }
                ) {
                    Icon(
                        imageVector = if (isLiked) Icons.Filled.Favorite
                                      else Icons.Outlined.FavoriteBorder,
                        contentDescription = if (isLiked) "Убрать лайк" else "Поставить лайк",
                        tint = iconColor,
                        modifier = Modifier
                            .size(28.dp)
                            .scale(iconScale)
                    )
                }

                Spacer(modifier = Modifier.width(4.dp))

                // Счётчик лайков (анимированное число)
                Text(
                    text = animatedCount.toString(),
                    style = MaterialTheme.typography.bodyLarge.copy(
                        fontWeight = if (isLiked) FontWeight.Bold else FontWeight.Normal,
                        fontSize = if (isLiked) 18.sp else 16.sp
                    ),
                    color = if (isLiked) Color(0xFFE91E63)
                            else MaterialTheme.colorScheme.onSurfaceVariant
                )
            }
        }
    }
}

// ─── Preview ─────────────────────────────────────────────────────────────────

@Preview(showBackground = true, backgroundColor = 0xFFF5F5F5)
@Composable
fun PostCardPreview() {
    MaterialTheme {
        Box(
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp)
        ) {
            PostCard(postNumber = 1)
        }
    }
}

```
build.gradle.kts
```
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.compose)
}

android {
    namespace = "com.example.examen_0103_harkov"
    compileSdk {
        version = release(36) {
            minorApiLevel = 1
        }
    }

    defaultConfig {
        applicationId = "com.example.examen_0103_harkov"
        minSdk = 24
        targetSdk = 36
        versionCode = 1
        versionName = "1.0"

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
            isMinifyEnabled = false
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_11
        targetCompatibility = JavaVersion.VERSION_11
    }
    buildFeatures {
        compose = true
    }
}

dependencies {
    implementation(platform(libs.androidx.compose.bom))
    implementation(libs.androidx.activity.compose)
    implementation(libs.androidx.compose.material3)
    implementation(libs.androidx.compose.ui)
    implementation(libs.androidx.compose.ui.graphics)
    implementation(libs.androidx.compose.ui.tooling.preview)
    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.lifecycle.runtime.ktx)
    implementation("androidx.compose.material:material-icons-extended")
    testImplementation(libs.junit)
    androidTestImplementation(platform(libs.androidx.compose.bom))
    androidTestImplementation(libs.androidx.compose.ui.test.junit4)
    androidTestImplementation(libs.androidx.espresso.core)
    androidTestImplementation(libs.androidx.junit)
    debugImplementation(libs.androidx.compose.ui.test.manifest)
    debugImplementation(libs.androidx.compose.ui.tooling)
}
```