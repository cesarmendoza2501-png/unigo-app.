# unigo-app.
app de carpooling


import streamlit as st
import pandas as pd
import time

# Configuración inicial de la página
st.set_page_config(page_title="UniGo by ByteCraft", page_icon="🚗", layout="wide")

# ==========================================
# CLASES DEL SISTEMA (POO)
# ==========================================

class UserManager:
    """Clase encargada de gestionar la sesión, autenticación y datos del usuario."""
    
    def __init__(self):
        if 'logged_in' not in st.session_state:
            st.session_state.logged_in = False
        if 'user_name' not in st.session_state:
            st.session_state.user_name = "Cesar Casas"
        if 'user_email' not in st.session_state:
            st.session_state.user_email = "cesar.casas@cun.edu.co"
        if 'user_role' not in st.session_state:
            st.session_state.user_role = "Conductor"
        if 'profile_pic' not in st.session_state:
            st.session_state.profile_pic = None
        if 'user_coins' not in st.session_state:
            st.session_state.user_coins = 350

    def login(self, email, password):
        if email.endswith("@cun.edu.co") and password:
            st.session_state.logged_in = True
            st.session_state.user_email = email
            st.session_state.user_name = email.split('@')[0].replace('.', ' ').title()
            return True
        return False

    def register(self, name, email, role, password, file):
        if email.endswith("@cun.edu.co") and password and file and name:
            st.session_state.logged_in = True
            st.session_state.user_name = name
            st.session_state.user_email = email
            st.session_state.user_role = role
            st.session_state.profile_pic = None
            return True
        return False

    def logout(self):
        st.session_state.logged_in = False
        st.rerun()

    def add_coins(self, amount):
        st.session_state.user_coins += amount


class TripManager:
    """Clase encargada de administrar los viajes compartidos, rutas y reservas."""
    
    def __init__(self):
        if 'trips' not in st.session_state:
            st.session_state.trips = [
                {
                    "id": 1,
                    "conductor": "Carlos Gómez",
                    "origen": "Calle 170",
                    "destino": "Sede Principal CUN",
                    "hora": "6:00 PM",
                    "asientos": 2,
                    "capacidad_total": 4,
                    "costo": 3500
                },
                {
                    "id": 2,
                    "conductor": "Ana María",
                    "origen": "Portal Norte",
                    "destino": "Sede Principal CUN",
                    "hora": "7:30 AM",
                    "asientos": 3,
                    "capacidad_total": 4,
                    "costo": 4000
                }
            ]

    def get_trips(self):
        return st.session_state.trips

    def add_trip(self, origen, destino, hora, asientos, costo, conductor_nombre):
        nuevo_viaje = {
            "id": len(st.session_state.trips) + 1,
            "conductor": conductor_nombre,
            "origen": origen,
            "destino": destino,
            "hora": hora,
            "asientos": asientos,
            "capacidad_total": asientos,
            "costo": costo
        }
        st.session_state.trips.append(nuevo_viaje)

    def update_trips(self, trips_list):
        st.session_state.trips = trips_list


class CarbonCalculator:
    """Clase orientada al cálculo científico de mitigación de gases de efecto invernadero (GEI)."""
    
    @staticmethod
    def calcular_ahorro_co2(distancia_km, factor_emision=0.192, ocupantes=2):
        """Calcula el CO2 ahorrado usando la fórmula del modelo."""
        emision_total = distancia_km * factor_emision
        emision_compartida = emision_total / ocupantes
        return emision_total - emision_compartida


class UniGoApp:
    """Clase controladora principal de la interfaz visual con Streamlit."""
    
    def __init__(self):
        self.user_manager = UserManager()
        self.trip_manager = TripManager()
        self.calculator = CarbonCalculator()

    def run(self):
        self.inject_styles()
        if not st.session_state.logged_in:
            self.render_auth_screen()
        else:
            self.render_main_dashboard()

    def inject_styles(self):
        st.markdown('''
        <style>
            .glass-card {
                background: rgba(255, 255, 255, 0.05);
                padding: 20px;
                border-radius: 15px;
                border: 1px solid rgba(255,255,255,0.1);
                margin-bottom: 15px;
            }
            .co2-panel {
                background: linear-gradient(135deg, #064e3b 0%, #10b981 100%);
                padding: 25px;
                border-radius: 20px;
                color: white;
                text-align: center;
                box-shadow: 0 10px 15px -3px rgba(16, 185, 129, 0.3);
            }
            .sos-alert {
                background: linear-gradient(135deg, #7f1d1d 0%, #dc2626 100%);
                padding: 20px;
                border-radius: 15px;
                color: white;
                text-align: center;
            }
        </style>
        ''', unsafe_allow_html=True)

    def render_auth_screen(self):
        col1, col2, col3 = st.columns([1, 2, 1])
        with col2:
            st.markdown("<h1 style='text-align: center;'>🚗 UniGo</h1>", unsafe_allow_html=True)
            st.markdown("<p style='text-align: center; color: gray;'>Movilidad Inteligente para la Comunidad CUN</p>", unsafe_allow_html=True)
            st.write("")
            
            tab_login, tab_register = st.tabs(["🔑 Ingresar", "📝 Registrar Usuario"])
            
            with tab_login:
                st.subheader("Iniciar Sesión")
                email_login = st.text_input("Correo Institucional", placeholder="tu.nombre@cun.edu.co", key="login_email")
                pass_login = st.text_input("Contraseña", type="password", key="login_pass")
                
                if st.button("Ingresar", type="primary", use_container_width=True):
                    if self.user_manager.login(email_login, pass_login):
                        st.success("Acceso concedido.")
                        time.sleep(1)
                        st.rerun()
                    elif not email_login.endswith("@cun.edu.co"):
                        st.error("Acceso denegado. Se requiere correo institucional @cun.edu.co")
                    else:
                        st.warning("Por favor ingresa tu correo y contraseña.")

            with tab_register:
                st.subheader("Crear Cuenta Nueva")
                new_name = st.text_input("Nombre y Apellido", placeholder="Ej: Juan Pérez")
                new_email = st.text_input("Correo Institucional (@cun.edu.co)", key="reg_email")
                new_role = st.radio("Rol Principal", ["Pasajero", "Conductor"], horizontal=True)
                new_pass = st.text_input("Crear Contraseña", type="password", key="reg_pass")
                uploaded_file = st.file_uploader("Sube tu recibo de matrícula vigente (PDF/JPG)", type=["pdf", "jpg", "jpeg"])
                
                if st.button("Registrar Usuario", type="primary", use_container_width=True):
                    if self.user_manager.register(new_name, new_email, new_role, new_pass, uploaded_file):
                        st.success("Registro exitoso. Redirigiendo...")
                        time.sleep(1.5)
                        st.rerun()
                    elif not new_email.endswith("@cun.edu.co"):
                        st.error("Registro inválido. Se requiere correo institucional de la CUN.")
                    else:
                        st.warning("Completa todos los campos y adjunta tu matrícula.")

    def render_main_dashboard(self):
        with st.sidebar:
            if st.session_state.profile_pic is not None:
                st.image(st.session_state.profile_pic, width=120)
            else:
                st.image(f"https://api.dicebear.com/7.x/avataaars/svg?seed={st.session_state.user_name}", width=120)
                
            st.write(f"### {st.session_state.user_name}")
            st.caption(f"{st.session_state.user_email}")
            st.write(f"**Rol:** {st.session_state.user_role}")
            st.info(f"🪙 **Eco-Puntos:** {st.session_state.user_coins} pts")
            st.divider()
            
            opcion_menu = st.radio(
                "Navegación",
                ["🚗 Mis Viajes y Rutas", "👤 Mi Perfil", "🌱 Impacto Ecológico", "🏆 Liderazgo CUN", "🚨 Seguridad SOS"]
            )
            
            st.divider()
            if st.button("Cerrar Sesión", use_container_width=True):
                self.user_manager.logout()

        if opcion_menu == "🚗 Mis Viajes y Rutas":
            self.render_trips_view()
        elif opcion_menu == "👤 Mi Perfil":
            self.render_profile_view()
        elif opcion_menu == "🌱 Impacto Ecológico":
            self.render_impact_view()
        elif opcion_menu == "🏆 Liderazgo CUN":
            self.render_leaderboard_view()
        elif opcion_menu == "🚨 Seguridad SOS":
            self.render_sos_view()

    def render_trips_view(self):
        st.title("🚗 Gestión de Rutas y Carpooling")
        col_rutas, col_accion = st.columns([2, 1])
        
        with col_rutas:
            st.subheader("📍 Rutas Disponibles en la Comunidad")
            trips = self.trip_manager.get_trips()
            
            if not trips:
                st.info("No hay rutas publicadas con asientos disponibles en este momento.")
            
            trips_to_keep = []
            for index, trip in enumerate(trips):
                st.markdown(f"""
                <div class="glass-card" style="border-left: 4px solid #3b82f6;">
                    <h4>🚗 Conductor: {trip['conductor']}</h4>
                    <p>🛣️ <b>Ruta:</b> {trip['origen']} hacia {trip['destino']}</p>
                    <p>⏰ <b>Hora:</b> {trip['hora']} | 💰 <b>Costo:</b> ${trip['costo']:,} COP por pasajero</p>
                    <p>👥 <b>Asientos libres:</b> {trip['asientos']} / {trip['capacidad_total']}</p>
                </div>
                """, unsafe_allow_html=True)
                
                col_btn1, col_btn2 = st.columns([1, 2])
                with col_btn1:
                    if st.button(f"Tomar Asiento", key=f"take_{index}"):
                        if trip['asientos'] > 1:
                            trip['asientos'] -= 1
                            self.user_manager.add_coins(50)
                            st.success("Asiento reservado con éxito. +50 Eco-Puntos acreditados 🪙.")
                            trips_to_keep.append(trip)
                            time.sleep(0.8)
                            st.rerun()
                        else:
                            st.success("Último asiento tomado. El viaje se ha completado.")
                            self.user_manager.add_coins(100)
                            time.sleep(1.2)
                            continue
                    else:
                        trips_to_keep.append(trip)
                st.write("")
            
            self.trip_manager.update_trips(trips_to_keep)

        with col_accion:
            if st.session_state.user_role == "Conductor":
                st.subheader("➕ Publicar Nueva Ruta")
                with st.form("publish_form"):
                    origen = st.text_input("Punto de Partida", placeholder="Ej: Portal Norte")
                    destino = st.text_input("Destino", placeholder="Sede Principal CUN")
                    hora = st.text_input("Hora de Salida", placeholder="Ej: 6:30 PM")
                    asientos = st.number_input("Asientos Disponibles", min_value=1, max_value=6, value=3)
                    costo = st.number_input("Costo por pasajero (COP)", min_value=0, value=3500, step=500)
                    
                    submitted = st.form_submit_button("Publicar Ruta", type="primary")
                    if submitted:
                        if origen and destino and hora:
                            self.trip_manager.add_trip(origen, destino, hora, asientos, costo, st.session_state.user_name)
                            st.success("Ruta publicada con éxito.")
                            time.sleep(1)
                            st.rerun()
                        else:
                            st.warning("Completa los campos principales.")
            else:
                st.subheader("💡 Panel de Pasajero")
                st.info("Explora las rutas activas y haz clic en 'Tomar Asiento' para unirte al trayecto y acumular Eco-Puntos.")

    def render_profile_view(self):
        st.title("👤 Configuración de Perfil y Logros")
        col_info, col_edit = st.columns([1, 1])
        
        with col_info:
            st.subheader("Datos Personales")
            st.write(f"**Nombre:** {st.session_state.user_name}")
            st.write(f"**Correo Institucional:** {st.session_state.user_email}")
            
            nuevo_rol = st.selectbox("Cambiar Rol en la App", ["Pasajero", "Conductor"], index=0 if st.session_state.user_role=="Pasajero" else 1)
            if nuevo_rol != st.session_state.user_role:
                st.session_state.user_role = nuevo_rol
                st.rerun()
                
            st.write("---")
            st.subheader("🏅 Insignias Desbloqueadas")
            st.markdown("- 🌍 **Eco-Conductor** (124 kg CO2 ahorrados)")
            st.markdown("- ⭐ **Puntualidad 5.0** (Calificación perfecta)")
            st.markdown("- 🌅 **Madrugador CUN** (Viajes antes de las 7:00 AM)")
            
            if st.session_state.user_role == "Conductor":
                st.write("---")
                st.subheader("🏍️ Vehículo Registrado")
                st.info("Modelo: **Yamaha R15 V3**\n\nSOAT: Vigente ✅")
                
        with col_edit:
            st.subheader("📸 Actualizar Foto de Perfil")
            nueva_foto = st.file_uploader("Sube una nueva imagen (JPG/PNG)", type=["png", "jpg", "jpeg"])
            if nueva_foto:
                if st.button("Guardar Foto", type="primary"):
                    st.session_state.profile_pic = nueva_foto
                    st.rerun()
            
            st.write("---")
            st.subheader("📊 Estadísticas de Actividad")
            st.metric(label="Calificación Promedio", value="4.9 / 5.0 ⭐")
            st.metric(label="Viajes Realizados", value="24")
            st.metric(label="Ahorro Económico Mensual", value="$180,000 COP", delta="vs transporte tradicional")

    def render_impact_view(self):
        st.title("🌱 Tu Impacto Ambiental y Económico")
        st.write("UniGo reduce la huella de carbono en Bogotá y optimiza el presupuesto de los estudiantes de la CUN.")
        
        col_metric, col_calc = st.columns([1, 1])
        
        with col_metric:
            st.markdown('<div class="co2-panel">', unsafe_allow_html=True)
            st.subheader("CO2 Ahorrado")
            st.title("124.5 kg")
            st.write("Equivale a plantar 6 árboles 🌳")
            st.progress(75, text="Progreso al nivel: Eco-Líder 🌍")
            st.markdown('</div>', unsafe_allow_html=True)
            
            st.write("")
            st.info("💰 **Ahorro Financiero Estimado:** Al compartir gastos en tus trayectos, te has ahorrado aproximadamente **$720,000 COP** en lo que va del semestre.")
            
            chart_data = pd.DataFrame({
                "Semanas": ["Semana 1", "Semana 2", "Semana 3", "Semana 4"],
                "CO2 Ahorrado (kg)": [20.5, 35.2, 45.0, 23.8]
            }).set_index("Semanas")
            st.bar_chart(chart_data, color="#10b981")
            
        with col_calc:
            st.subheader("📐 Modelo de Cálculo de Emisiones")
            st.info("Metodología oficial de mitigación de gases de efecto invernadero (GEI).")
            
            st.latex(r"Ahorro \ CO_2 = (D \times F) - \left(\frac{D \times F}{N}\right)")
            
            st.write("""
            **Desglose:**
            - **$D$ (Distancia):** Kilómetros recorridos hacia la CUN.
            - **$F$ (Factor de Emisión):** **0.192 kg de CO2/km** por vehículo de combustión promedio en Bogotá.
            - **$N$ (Número de Ocupantes):** División de emisiones entre los ocupantes del automóvil.
            """)
            st.warning("💡 **Nota:** Compartir vehículo reduce las emisiones del trayecto en un 75%.")

    def render_leaderboard_view(self):
        st.title("🏆 Ranking Ecológico Institucional")
        st.markdown("Estudiantes de la CUN que lideran el impacto en movilidad sostenible este mes.")
        
        leaderboard_data = pd.DataFrame({
            "Puesto": ["🥇 1", "🥈 2", "🥉 3 (Tú)", "4", "5"],
            "Estudiante": ["Carlos Gómez (Ing. Sistemas)", "Ana María (Administración)", st.session_state.user_name, "Mateo Rodríguez", "Laura Sofia"],
            "CO2 Ahorrado": ["185.2 kg", "150.8 kg", "124.5 kg", "98.0 kg", "85.4 kg"],
            "Eco-Puntos": [620, 510, st.session_state.user_coins, 380, 310]
        })
        
        st.table(leaderboard_data)
        st.success("🎉 Sigue compartiendo viajes para escalar posiciones y obtener beneficios institucionales.")

    def render_sos_view(self):
        st.title("🚨 Centro de Seguridad y Emergencias")
        st.write("Módulo de protección para la integridad en los trayectos urbanos.")
        
        st.markdown('''
        <div class="sos-alert">
            <h3>⚠️ BOTÓN DE PÁNICO RÁPIDO</h3>
            <p>Al activar este mecanismo, se notificará de inmediato al soporte técnico y a tu contacto de emergencia registrado.</p>
        </div>
        ''', unsafe_allow_html=True)
        
        st.write("")
        col_sos1, col_sos2 = st.columns(2)
        
        with col_sos1:
            if st.button("🚨 ACTIVAR ALERTA SOS", use_container_width=True, type="primary"):
                st.error("Alerta emitida. Ubicación compartida con el equipo de seguridad institucional.")
                
        with col_sos2:
            if st.button("📍 Compartir Ubicación en Tiempo Real", use_container_width=True):
                st.success("Enlace de rastreo seguro generado y copiado al portapapeles.")
                
        st.write("---")
        st.subheader("📞 Contactos de Confianza")
        st.text_input("Número de contacto de emergencia", value="310 456 7890")
        st.button("Guardar Contacto")


# ==========================================
# EJECUCIÓN DE LA APLICACIÓN
# ==========================================
if __name__ == "__main__":
    app = UniGoApp()
    app.run()
