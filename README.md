# Calculadora-Concreto
import streamlit as st

st.set_page_config(page_title="Calculadora de Concreto", layout="centered")

st.title("🧱 Calculadora de Volume de Concreto")
st.caption("Laje • Viga • Pilar • Soma geral • Perdas")

def laje(area_m2: float, esp_cm: float) -> float:
    return area_m2 * (esp_cm / 100.0)

def viga(larg_cm: float, alt_cm: float, comp_m: float) -> float:
    return (larg_cm / 100.0) * (alt_cm / 100.0) * comp_m

def pilar(larg_cm: float, prof_cm: float, alt_m: float) -> float:
    return (larg_cm / 100.0) * (prof_cm / 100.0) * alt_m

# Estado para guardar itens adicionados
if "itens" not in st.session_state:
    st.session_state.itens = []

perda_pct = st.slider("Perda (%)", min_value=0.0, max_value=20.0, value=5.0, step=0.5)

st.divider()

tipo = st.selectbox("Selecione o elemento", ["Laje", "Viga", "Pilar"])

with st.form("form_concreto", clear_on_submit=False):
    if tipo == "Laje":
        area = st.number_input("Área (m²)", min_value=0.0, value=43.0, step=1.0)
        esp = st.number_input("Espessura (cm)", min_value=0.0, value=5.0, step=0.5)
        qtd = st.number_input("Quantidade", min_value=1, value=1, step=1)
    elif tipo == "Viga":
        larg = st.number_input("Largura (cm)", min_value=0.0, value=12.0, step=1.0)
        alt = st.number_input("Altura (cm)", min_value=0.0, value=30.0, step=1.0)
        comp = st.number_input("Comprimento (m)", min_value=0.0, value=3.0, step=0.1)
        qtd = st.number_input("Quantidade", min_value=1, value=1, step=1)
    else:  # Pilar
        larg = st.number_input("Largura (cm)", min_value=0.0, value=20.0, step=1.0)
        prof = st.number_input("Profundidade (cm)", min_value=0.0, value=20.0, step=1.0)
        alt = st.number_input("Altura (m)", min_value=0.0, value=3.0, step=0.1)
        qtd = st.number_input("Quantidade", min_value=1, value=1, step=1)

    add = st.form_submit_button("➕ Adicionar à lista")
    calc = st.form_submit_button("🧮 Calcular agora (sem adicionar)")

def calcular_volume_atual():
    if tipo == "Laje":
        vol_unit = laje(area, esp)
        desc = f"Laje: {area:.2f} m² x {esp:.2f} cm"
    elif tipo == "Viga":
        vol_unit = viga(larg, alt, comp)
        desc = f"Viga: {larg:.1f}x{alt:.1f} cm x {comp:.2f} m"
    else:
        vol_unit = pilar(larg, prof, alt)
        desc = f"Pilar: {larg:.1f}x{prof:.1f} cm x {alt:.2f} m"

    vol_total = vol_unit * int(qtd)
    return desc, vol_unit, vol_total

if add or calc:
    desc, vol_unit, vol_total = calcular_volume_atual()

    st.success(f"Volume unitário: **{vol_unit:.3f} m³**  |  Volume (qtd {int(qtd)}): **{vol_total:.3f} m³**")

    if add:
        st.session_state.itens.append({
            "Elemento": tipo,
            "Descrição": desc,
            "Qtd": int(qtd),
            "Vol. unitário (m³)": round(vol_unit, 4),
            "Vol. total (m³)": round(vol_total, 4),
        })

st.divider()
st.subheader("📋 Lista de elementos")

if st.session_state.itens:
    st.dataframe(st.session_state.itens, use_container_width=True)

    soma = sum(item["Vol. total (m³)"] for item in st.session_state.itens)
    perda = soma * (perda_pct / 100.0)
    total = soma + perda

    c1, c2, c3 = st.columns(3)
    c1.metric("Volume estrutural", f"{soma:.3f} m³")
    c2.metric(f"Perda ({perda_pct:.1f}%)", f"{perda:.3f} m³")
    c3.metric("Total a pedir", f"{total:.3f} m³")

    colA, colB = st.columns(2)
    with colA:
        if st.button("🧹 Limpar lista"):
            st.session_state.itens = []
            st.rerun()
    with colB:
        st.download_button(
            "⬇️ Baixar CSV",
            data=("Elemento,Descrição,Qtd,Vol. unitário (m³),Vol. total (m³)\n" +
                  "\n".join(f"{i['Elemento']},{i['Descrição']},{i['Qtd']},{i['Vol. unitário (m³)']},{i['Vol. total (m³)']}"
                            for i in st.session_state.itens)),
            file_name="volumes_concreto.csv",
            mime="text/csv"
        )
else:
    st.info("Adicione elementos para somar volumes e gerar o total com perda.")
