---
tags:
  - Java
  - Patrones
  - Estructural
Creado: 2026-05-21
Relacionado:
  - Patrones de Diseño
Palabras clave:
  - facade, simplificación, subsistema, interfaz unificada
---

## 🧠 Concepto

Proporciona una **interfaz simplificada** para un subsistema complejo. Oculta la complejidad interna.

**Cuándo usarlo:** APIs complejas de terceros, módulos con muchas clases, simplificar capas de la aplicación.

## Implementación

```java
// Subsistema complejo
public class SistemaDeVideo {
    public void cargar(String archivo) { System.out.println("Cargando video: " + archivo); }
    public void decodificar() { System.out.println("Decodificando..."); }
    public void renderizar() { System.out.println("Renderizando..."); }
    public void reproducir() { System.out.println("Reproduciendo..."); }
}

public class SistemaDeAudio {
    public void cargarAudio(String archivo) { System.out.println("Cargando audio: " + archivo); }
    public void sincronizar() { System.out.println("Sincronizando A/V..."); }
    public void reproducirAudio() { System.out.println("Reproduciendo audio..."); }
}

public class SistemaDeSubtitulos {
    public void cargarSubtitulos(String idioma) { System.out.println("Cargando subtítulos: " + idioma); }
    public void mostrar() { System.out.println("Mostrando subtítulos..."); }
}

// Facade — interfaz simple
public class ReproductorMediaFacade {
    private final SistemaDeVideo video;
    private final SistemaDeAudio audio;
    private final SistemaDeSubtitulos subtitulos;

    public ReproductorMediaFacade() {
        this.video = new SistemaDeVideo();
        this.audio = new SistemaDeAudio();
        this.subtitulos = new SistemaDeSubtitulos();
    }

    // Un solo método para toda la complejidad
    public void reproducir(String archivo, String idiomaSubs) {
        video.cargar(archivo);
        video.decodificar();
        audio.cargarAudio(archivo);
        audio.sincronizar();
        subtitulos.cargarSubtitulos(idiomaSubs);
        video.renderizar();
        subtitulos.mostrar();
        video.reproducir();
        audio.reproducirAudio();
    }
}

// Uso — el cliente no necesita saber de los subsistemas
ReproductorMediaFacade reproductor = new ReproductorMediaFacade();
reproductor.reproducir("pelicula.mp4", "español");
```

## Facade en aplicaciones reales

```java
// Facade de servicio — orquesta múltiples servicios
@Service
public class RegistroUsuarioFacade {
    private final UsuarioService usuarioService;
    private final EmailService emailService;
    private final AuditService auditService;
    private final PerfilService perfilService;

    public UsuarioResponse registrar(RegistroRequest request) {
        // 1. Crear usuario
        Usuario usuario = usuarioService.crear(request);
        // 2. Crear perfil por defecto
        perfilService.crearPerfilDefecto(usuario.getId());
        // 3. Enviar email de bienvenida
        emailService.enviarBienvenida(usuario.getEmail());
        // 4. Auditar
        auditService.registrar("REGISTRO", usuario.getId());
        // 5. Retornar respuesta simplificada
        return UsuarioResponse.from(usuario);
    }
}
```

## 📌 Notas Rápidas
