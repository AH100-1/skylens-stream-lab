# 왜곡 핀홀 투영과 야코비안

## 모델
세계 점 X, 자세 (R, t): Xc = R X + t. 정규 좌표 n = (Xc.x/Xc.z, Xc.y/Xc.z).
r² = x² + y², ρ = 1 + k1 r² + k2 r⁴ (Brown 1966 계열 방사·접선 모델)

    xd = x ρ + 2 p1 x y + p2 (r² + 2x²)
    yd = y ρ + p1 (r² + 2y²) + 2 p2 x y
    u = fx xd + cx,  v = fy yd + cy

## ∂(xd, yd)/∂(x, y)
∂ρ/∂x = 2x ρ',  ∂ρ/∂y = 2y ρ',  ρ' = ∂ρ/∂r² = k1 + 2 k2 r².

    ∂xd/∂x = ρ + 2x² ρ' + 2 p1 y + 6 p2 x
    ∂xd/∂y = 2xy ρ' + 2 p1 x + 2 p2 y
    ∂yd/∂x = 2xy ρ' + 2 p1 x + 2 p2 y
    ∂yd/∂y = ρ + 2y² ρ' + 6 p1 y + 2 p2 x

## ∂(xd, yd)/∂(k1, k2, p1, p2)

    [ x r²   x r⁴   2xy        r² + 2x² ]
    [ y r²   y r⁴   r² + 2y²   2xy      ]

## 연쇄
∂n/∂Xc = [[1/z, 0, −x/z²], [0, 1/z, −y/z²]] (여기서 x, y 는 Xc 성분).
∂(u,v)/∂Xc = diag(fx, fy) · ∂d/∂n · ∂n/∂Xc.

- 세계 점: ∂Xc/∂X = R.
- 자세(왼쪽 섭동 R ← exp([ω]×) R): Xc(ω) ≈ (I + [ω]×) R X + t 이므로 ∂Xc/∂ω = −[R X]×, ∂Xc/∂t = I.
- 내부: ∂u/∂fx = xd, ∂v/∂fy = yd, ∂u/∂cx = ∂v/∂cy = 1, 왜곡 계수는 diag(fx, fy) · 위 2×4 행렬.

## 역왜곡
g(n) = distort(n) − nd = 0 을 뉴턴법 n ← n − (∂d/∂n)⁻¹ g(n) 으로 푼다. 초기값 n = nd.
