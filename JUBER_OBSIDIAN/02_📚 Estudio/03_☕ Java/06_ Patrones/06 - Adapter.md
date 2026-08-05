---
tags:
  - Java
  - Patrones
  - Estructural
Creado: 2026-05-21
Relacionado:
  - Patrones de Diseño
Palabras clave:
  - adapter, wrapper, compatibilidad, interfaz incompatible
---

## 🧠 Concepto

Permite que clases con interfaces **incompatibles** trabajen juntas. Actúa como un traductor entre dos interfaces.

**Cuándo usarlo:** Integrar código legado, usar librerías de terceros, cuando no puedes modificar la clase existente.

## Implementación

```java
// Lo que el cliente espera
public interface ReproductorMusica {
    void reproducir(String formato, String archivo);
}

// Clase existente que no puedes modificar
public class ReproductorMp4 {
    public void playMp4(String archivo) {
        System.out.println("Reproduciendo MP4: " + archivo);
    }
}

public class ReproductorVlc {
    public void playVlc(String archivo) {
        System.out.println("Reproduciendo VLC: " + archivo);
    }
}

// Adapter — traduce la interfaz esperada a la clase existente
public class MediaAdapter implements ReproductorMusica {
    private ReproductorMp4 mp4Player;
    private ReproductorVlc vlcPlayer;

    @Override
    public void reproducir(String formato, String archivo) {
        switch (formato.toLowerCase()) {
            case "mp4" -> {
                if (mp4Player == null) mp4Player = new ReproductorMp4();
                mp4Player.playMp4(archivo);
            }
            case "vlc" -> {
                if (vlcPlayer == null) vlcPlayer = new ReproductorVlc();
                vlcPlayer.playVlc(archivo);
            }
            default -> System.out.println("Formato no soportado: " + formato);
        }
    }
}

// Cliente
public class ReproductorAudio implements ReproductorMusica {
    private MediaAdapter adapter;

    @Override
    public void reproducir(String formato, String archivo) {
        if (formato.equals("mp3")) {
            System.out.println("Reproduciendo MP3: " + archivo);
        } else {
            adapter = new MediaAdapter();
            adapter.reproducir(formato, archivo);
        }
    }
}
```

## Ejemplo real — adaptando API externa

```java
// API que tenemos de terceros
public class PasarelaPagoLegacy {
    public String procesarPago(double monto, String cuenta, String token) { ... }
}

// Interfaz que quiere usar nuestra app
public interface PasarelaPago {
    PagoResult pagar(PagoRequest request);
}

// Adapter
public class PasarelaPagoAdapter implements PasarelaPago {
    private final PasarelaPagoLegacy legacy;

    public PasarelaPagoAdapter(PasarelaPagoLegacy legacy) {
        this.legacy = legacy;
    }

    @Override
    public PagoResult pagar(PagoRequest request) {
        String respuesta = legacy.procesarPago(
            request.getMonto(),
            request.getCuenta(),
            request.getToken()
        );
        return PagoResult.from(respuesta);
    }
}
```

## 📌 Notas Rápidas
