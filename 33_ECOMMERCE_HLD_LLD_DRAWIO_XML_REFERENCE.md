# E-Commerce App HLD & LLD (draw.io Integration Reference)

---

## 1. HLD: High-Level Design (draw.io)

### 1.1 HLD Diagram (draw.io XML)
- Use this XML in draw.io: File → Import From → Device, paste XML.
- Each component is labeled and commented for interview reference.

```xml
<!-- HLD: E-Commerce System -->
<mxfile host="app.diagrams.net">
  <diagram name="HLD">
    <mxGraphModel dx="1000" dy="1000" grid="1" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="1" pageScale="1" pageWidth="827" pageHeight="1169" math="0" shadow="0">
      <root>
        <mxCell id="0" />
        <mxCell id="1" parent="0" />
        <!-- User Service -->
        <mxCell id="2" value="User Service" style="rounded=1;whiteSpace=wrap;html=1;fillColor=#d5e8d4;" vertex="1" parent="1">
          <mxGeometry x="60" y="60" width="120" height="60" as="geometry" />
        </mxCell>
        <!-- Product Service -->
        <mxCell id="3" value="Product Service" style="rounded=1;whiteSpace=wrap;html=1;fillColor=#d5e8d4;" vertex="1" parent="1">
          <mxGeometry x="260" y="60" width="120" height="60" as="geometry" />
        </mxCell>
        <!-- Order Service -->
        <mxCell id="4" value="Order Service" style="rounded=1;whiteSpace=wrap;html=1;fillColor=#d5e8d4;" vertex="1" parent="1">
          <mxGeometry x="160" y="200" width="120" height="60" as="geometry" />
        </mxCell>
        <!-- User DB -->
        <mxCell id="5" value="User DB" style="shape=cylinder;whiteSpace=wrap;html=1;fillColor=#dae8fc;" vertex="1" parent="1">
          <mxGeometry x="60" y="140" width="80" height="40" as="geometry" />
        </mxCell>
        <!-- Product DB -->
        <mxCell id="6" value="Product DB" style="shape=cylinder;whiteSpace=wrap;html=1;fillColor=#dae8fc;" vertex="1" parent="1">
          <mxGeometry x="300" y="140" width="80" height="40" as="geometry" />
        </mxCell>
        <!-- Order DB -->
        <mxCell id="7" value="Order DB" style="shape=cylinder;whiteSpace=wrap;html=1;fillColor=#dae8fc;" vertex="1" parent="1">
          <mxGeometry x="200" y="280" width="80" height="40" as="geometry" />
        </mxCell>
        <!-- Payment Gateway -->
        <mxCell id="8" value="Payment Gateway" style="rounded=1;whiteSpace=wrap;html=1;fillColor=#fff2cc;" vertex="1" parent="1">
          <mxGeometry x="200" y="360" width="120" height="40" as="geometry" />
        </mxCell>
        <!-- Connections -->
        <mxCell id="9" style="edgeStyle=orthogonalEdgeStyle;endArrow=block;html=1;" edge="1" parent="1" source="2" target="5">
          <mxGeometry relative="1" as="geometry" />
        </mxCell>
        <mxCell id="10" style="edgeStyle=orthogonalEdgeStyle;endArrow=block;html=1;" edge="1" parent="1" source="3" target="6">
          <mxGeometry relative="1" as="geometry" />
        </mxCell>
        <mxCell id="11" style="edgeStyle=orthogonalEdgeStyle;endArrow=block;html=1;" edge="1" parent="1" source="4" target="7">
          <mxGeometry relative="1" as="geometry" />
        </mxCell>
        <mxCell id="12" style="edgeStyle=orthogonalEdgeStyle;endArrow=block;html=1;" edge="1" parent="1" source="4" target="8">
          <mxGeometry relative="1" as="geometry" />
        </mxCell>
        <mxCell id="13" style="edgeStyle=orthogonalEdgeStyle;endArrow=block;html=1;" edge="1" parent="1" source="2" target="4">
          <mxGeometry relative="1" as="geometry" />
        </mxCell>
        <mxCell id="14" style="edgeStyle=orthogonalEdgeStyle;endArrow=block;html=1;" edge="1" parent="1" source="3" target="4">
          <mxGeometry relative="1" as="geometry" />
        </mxCell>
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

**Interview Comments:**
- Each service is stateless and horizontally scalable.
- DBs are separated for isolation and scaling.
- Payment Gateway is external and integrated via secure API.
- Arrows show data and request flow.

---

## 2. LLD: Low-Level Design (draw.io)

### 2.1 LLD Diagram (draw.io XML)
- Use this XML in draw.io for a sequence diagram of order placement.

```xml
<!-- LLD: Order Placement Sequence -->
<mxfile host="app.diagrams.net">
  <diagram name="LLD">
    <mxGraphModel dx="1000" dy="1000" grid="1" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="1" pageScale="1" pageWidth="827" pageHeight="1169" math="0" shadow="0">
      <root>
        <mxCell id="0" />
        <mxCell id="1" parent="0" />
        <!-- Lifelines -->
        <mxCell id="2" value="User" style="umlLifeline;" vertex="1" parent="1">
          <mxGeometry x="60" y="60" width="40" height="300" as="geometry" />
        </mxCell>
        <mxCell id="3" value="Order Service" style="umlLifeline;" vertex="1" parent="1">
          <mxGeometry x="160" y="60" width="40" height="300" as="geometry" />
        </mxCell>
        <mxCell id="4" value="Product Service" style="umlLifeline;" vertex="1" parent="1">
          <mxGeometry x="260" y="60" width="40" height="300" as="geometry" />
        </mxCell>
        <mxCell id="5" value="Payment Gateway" style="umlLifeline;" vertex="1" parent="1">
          <mxGeometry x="360" y="60" width="40" height="300" as="geometry" />
        </mxCell>
        <!-- Messages -->
        <mxCell id="6" value="POST /orders" style="endArrow=block;html=1;" edge="1" parent="1" source="2" target="3">
          <mxGeometry relative="1" as="geometry" />
        </mxCell>
        <mxCell id="7" value="GET /products/:id" style="endArrow=block;html=1;" edge="1" parent="1" source="3" target="4">
          <mxGeometry relative="1" as="geometry" />
        </mxCell>
        <mxCell id="8" value="POST /pay" style="endArrow=block;html=1;" edge="1" parent="1" source="3" target="5">
          <mxGeometry relative="1" as="geometry" />
        </mxCell>
        <mxCell id="9" value="Save Order" style="endArrow=block;html=1;dashed=1;" edge="1" parent="1" source="3" target="3">
          <mxGeometry relative="1" as="geometry" />
        </mxCell>
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

**Interview Comments:**
- Sequence shows order placement: User → Order Service → Product Service → Payment Gateway.
- Dashed arrow for DB write.
- Each message is labeled with the API call.

---

## 3. Reference Rules for draw.io Diagrams
- Label every component and connection.
- Use consistent shapes/colors for services, DBs, and externals.
- Arrows must indicate direction and type (solid for sync, dashed for async/error).
- Add comments/notes for interview talking points.
- Use layers for HLD/LLD separation.
- Export as PNG/SVG for sharing.

---

# Use these XMLs in draw.io for instant, interview-ready HLD/LLD with comments!
