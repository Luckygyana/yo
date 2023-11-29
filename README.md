<h1>Approach</h1>
<ol>
<li>Clustering</li>
<li>Optimization</li>
<li>Smoothening</li>
</ol>
<h5 id="clustering">Clustering </h5>
<ol>
<li>
<p>The Gap Threshold (G):<br>
This is a user-defined parameter (<code>gap_param</code> with a default value <code>1.3</code>), that acts as a threshold to decide whether two consecutive dialogues belong to the same cluster or different clusters. It's a measure of temporal distance in seconds.</p>
</li>
<li>
<p>Clustering Logic:<br>
The algorithm sorts voiceover dialogues into groups (clusters) where each cluster consists of dialogues that follow each other with a time gap smaller than or equal to the gap threshold (G).</p>
</li>
</ol>
<p><strong>Mathematically, the clustering can be defined as follows:</strong></p>
<ul>
<li>Let <code>D = [d0, d1, ..., dn]</code> be a list of dialogues, where each <code>di</code> has <code>start_time</code> and <code>end_time</code>.</li>
<li>A pair of dialogues <code>(di, dj)</code> belongs to the same cluster if <code>dj.start_time - di.end_time &lt;= G</code>, where <code>i &lt; j</code>.</li>
<li>If <code>dj.start_time - di.end_time &gt; G</code>, then <code>dj</code> starts a new cluster.</li>
</ul>
<h5 id="optimization">Optimization </h5>
<p>Objective Function:<br>
The objective function to be minimized is defined as the sum of the squares of the differences between subsequent variables in a list. For a list of variables <code>vars</code> with N elements, the objective function <code>f(vars)</code> can be written as:<br>
<span class="katex-display"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><semantics><mrow><mi>f</mi><mo stretchy="false">(</mo><mi>v</mi><mi>a</mi><mi>r</mi><mi>s</mi><mo stretchy="false">)</mo><mo>=</mo><munderover><mo>∑</mo><mrow><mi>i</mi><mo>=</mo><mn>0</mn></mrow><mrow><mi>N</mi><mo>−</mo><mn>2</mn></mrow></munderover><mo stretchy="false">(</mo><mi>v</mi><mi>a</mi><mi>r</mi><mi>s</mi><mo stretchy="false">[</mo><mi>i</mi><mo>+</mo><mn>1</mn><mo stretchy="false">]</mo><mo>−</mo><mi>v</mi><mi>a</mi><mi>r</mi><mi>s</mi><mo stretchy="false">[</mo><mi>i</mi><mo stretchy="false">]</mo><msup><mo stretchy="false">)</mo><mn>2</mn></msup></mrow><annotation encoding="application/x-tex">f(vars) = \sum_{i=0}^{N-2} (vars[i+1] - vars[i])^2</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height:1em;vertical-align:-0.25em;"></span><span class="mord mathnormal" style="margin-right:0.10764em;">f</span><span class="mopen">(</span><span class="mord mathnormal" style="margin-right:0.03588em;">v</span><span class="mord mathnormal">a</span><span class="mord mathnormal">rs</span><span class="mclose">)</span><span class="mspace" style="margin-right:0.2778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right:0.2778em;"></span></span><span class="base"><span class="strut" style="height:3.106em;vertical-align:-1.2777em;"></span><span class="mop op-limits"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height:1.8283em;"><span style="top:-1.8723em;margin-left:0em;"><span class="pstrut" style="height:3.05em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathnormal mtight">i</span><span class="mrel mtight">=</span><span class="mord mtight">0</span></span></span></span><span style="top:-3.05em;"><span class="pstrut" style="height:3.05em;"></span><span><span class="mop op-symbol large-op">∑</span></span></span><span style="top:-4.3em;margin-left:0em;"><span class="pstrut" style="height:3.05em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathnormal mtight" style="margin-right:0.10903em;">N</span><span class="mbin mtight">−</span><span class="mord mtight">2</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height:1.2777em;"><span></span></span></span></span></span><span class="mopen">(</span><span class="mord mathnormal" style="margin-right:0.03588em;">v</span><span class="mord mathnormal">a</span><span class="mord mathnormal">rs</span><span class="mopen">[</span><span class="mord mathnormal">i</span><span class="mspace" style="margin-right:0.2222em;"></span><span class="mbin">+</span><span class="mspace" style="margin-right:0.2222em;"></span></span><span class="base"><span class="strut" style="height:1em;vertical-align:-0.25em;"></span><span class="mord">1</span><span class="mclose">]</span><span class="mspace" style="margin-right:0.2222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right:0.2222em;"></span></span><span class="base"><span class="strut" style="height:1.1141em;vertical-align:-0.25em;"></span><span class="mord mathnormal" style="margin-right:0.03588em;">v</span><span class="mord mathnormal">a</span><span class="mord mathnormal">rs</span><span class="mopen">[</span><span class="mord mathnormal">i</span><span class="mclose">]</span><span class="mclose"><span class="mclose">)</span><span class="msupsub"><span class="vlist-t"><span class="vlist-r"><span class="vlist" style="height:0.8641em;"><span style="top:-3.113em;margin-right:0.05em;"><span class="pstrut" style="height:2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight">2</span></span></span></span></span></span></span></span></span></span></span></span><br>
The goal of the optimization is to find the values of <code>vars</code> that minimize this objective function.</p>
<p>Equality Constraint:<br>
The problem includes an equality constraint which is the sum of a series of fractions involving the variables <code>L[i]</code> and <code>vars[i]</code> subtracted by a constant series involving <code>L[i]</code> and <code>A[i]</code>. The equality constraint <code>g(vars)</code> must be satisfied such that:<br>
<span class="katex-display"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><semantics><mrow><mi>g</mi><mo stretchy="false">(</mo><mi>v</mi><mi>a</mi><mi>r</mi><mi>s</mi><mo stretchy="false">)</mo><mo>=</mo><munderover><mo>∑</mo><mrow><mi>i</mi><mo>=</mo><mn>0</mn></mrow><mrow><mi>N</mi><mo>−</mo><mn>1</mn></mrow></munderover><mfrac><mrow><mi>L</mi><mo stretchy="false">[</mo><mi>i</mi><mo stretchy="false">]</mo></mrow><mrow><mi>v</mi><mi>a</mi><mi>r</mi><mi>s</mi><mo stretchy="false">[</mo><mi>i</mi><mo stretchy="false">]</mo></mrow></mfrac><mo>−</mo><munderover><mo>∑</mo><mrow><mi>i</mi><mo>=</mo><mn>0</mn></mrow><mrow><mi>N</mi><mo>−</mo><mn>1</mn></mrow></munderover><mfrac><mrow><mi>L</mi><mo stretchy="false">[</mo><mi>i</mi><mo stretchy="false">]</mo></mrow><mrow><mi>A</mi><mo stretchy="false">[</mo><mi>i</mi><mo stretchy="false">]</mo></mrow></mfrac><mo>=</mo><mn>0</mn></mrow><annotation encoding="application/x-tex">g(vars) = \sum_{i=0}^{N-1} \frac{L[i]}{vars[i]} - \sum_{i=0}^{N-1} \frac{L[i]}{A[i]} = 0</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height:1em;vertical-align:-0.25em;"></span><span class="mord mathnormal" style="margin-right:0.03588em;">g</span><span class="mopen">(</span><span class="mord mathnormal" style="margin-right:0.03588em;">v</span><span class="mord mathnormal">a</span><span class="mord mathnormal">rs</span><span class="mclose">)</span><span class="mspace" style="margin-right:0.2778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right:0.2778em;"></span></span><span class="base"><span class="strut" style="height:3.106em;vertical-align:-1.2777em;"></span><span class="mop op-limits"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height:1.8283em;"><span style="top:-1.8723em;margin-left:0em;"><span class="pstrut" style="height:3.05em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathnormal mtight">i</span><span class="mrel mtight">=</span><span class="mord mtight">0</span></span></span></span><span style="top:-3.05em;"><span class="pstrut" style="height:3.05em;"></span><span><span class="mop op-symbol large-op">∑</span></span></span><span style="top:-4.3em;margin-left:0em;"><span class="pstrut" style="height:3.05em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathnormal mtight" style="margin-right:0.10903em;">N</span><span class="mbin mtight">−</span><span class="mord mtight">1</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height:1.2777em;"><span></span></span></span></span></span><span class="mspace" style="margin-right:0.1667em;"></span><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height:1.427em;"><span style="top:-2.314em;"><span class="pstrut" style="height:3em;"></span><span class="mord"><span class="mord mathnormal" style="margin-right:0.03588em;">v</span><span class="mord mathnormal">a</span><span class="mord mathnormal">rs</span><span class="mopen">[</span><span class="mord mathnormal">i</span><span class="mclose">]</span></span></span><span style="top:-3.23em;"><span class="pstrut" style="height:3em;"></span><span class="frac-line" style="border-bottom-width:0.04em;"></span></span><span style="top:-3.677em;"><span class="pstrut" style="height:3em;"></span><span class="mord"><span class="mord mathnormal">L</span><span class="mopen">[</span><span class="mord mathnormal">i</span><span class="mclose">]</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height:0.936em;"><span></span></span></span></span></span><span class="mclose nulldelimiter"></span></span><span class="mspace" style="margin-right:0.2222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right:0.2222em;"></span></span><span class="base"><span class="strut" style="height:3.106em;vertical-align:-1.2777em;"></span><span class="mop op-limits"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height:1.8283em;"><span style="top:-1.8723em;margin-left:0em;"><span class="pstrut" style="height:3.05em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathnormal mtight">i</span><span class="mrel mtight">=</span><span class="mord mtight">0</span></span></span></span><span style="top:-3.05em;"><span class="pstrut" style="height:3.05em;"></span><span><span class="mop op-symbol large-op">∑</span></span></span><span style="top:-4.3em;margin-left:0em;"><span class="pstrut" style="height:3.05em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathnormal mtight" style="margin-right:0.10903em;">N</span><span class="mbin mtight">−</span><span class="mord mtight">1</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height:1.2777em;"><span></span></span></span></span></span><span class="mspace" style="margin-right:0.1667em;"></span><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height:1.427em;"><span style="top:-2.314em;"><span class="pstrut" style="height:3em;"></span><span class="mord"><span class="mord mathnormal">A</span><span class="mopen">[</span><span class="mord mathnormal">i</span><span class="mclose">]</span></span></span><span style="top:-3.23em;"><span class="pstrut" style="height:3em;"></span><span class="frac-line" style="border-bottom-width:0.04em;"></span></span><span style="top:-3.677em;"><span class="pstrut" style="height:3em;"></span><span class="mord"><span class="mord mathnormal">L</span><span class="mopen">[</span><span class="mord mathnormal">i</span><span class="mclose">]</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height:0.936em;"><span></span></span></span></span></span><span class="mclose nulldelimiter"></span></span><span class="mspace" style="margin-right:0.2778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right:0.2778em;"></span></span><span class="base"><span class="strut" style="height:0.6444em;"></span><span class="mord">0</span></span></span></span></span><br>
This means that the sum of <code>L[i]/vars[i]</code> over all elements must equal the sum of <code>L[i]/A[i]</code> over all elements.</p>
<p>Inequality Constraints:<br>
There are also inequality constraints which are expressed as a sequence of inequalities that relate consecutive variables based on the parameters <code>L</code>, <code>A</code>, and <code>R</code>. The inequality constraints <code>h(vars)</code> must be such that:<br>
<span class="katex-display"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><semantics><mrow><msub><mi>h</mi><mi>i</mi></msub><mo stretchy="false">(</mo><mi>v</mi><mi>a</mi><mi>r</mi><mi>s</mi><mo stretchy="false">)</mo><mo>=</mo><mrow><mo fence="true">{</mo><mtable rowspacing="0.16em" columnalign="left left" columnspacing="1em"><mtr><mtd><mstyle scriptlevel="0" displaystyle="false"><mrow><mi>R</mi><mo>+</mo><mrow><mo fence="true">(</mo><mfrac><mrow><mi>L</mi><mo stretchy="false">[</mo><mn>0</mn><mo stretchy="false">]</mo></mrow><mrow><mi>v</mi><mi>a</mi><mi>r</mi><mi>s</mi><mo stretchy="false">[</mo><mn>0</mn><mo stretchy="false">]</mo></mrow></mfrac><mo>−</mo><mfrac><mrow><mi>L</mi><mo stretchy="false">[</mo><mn>0</mn><mo stretchy="false">]</mo></mrow><mrow><mi>A</mi><mo stretchy="false">[</mo><mn>0</mn><mo stretchy="false">]</mo></mrow></mfrac><mo fence="true">)</mo></mrow><mo>≥</mo><mn>0</mn></mrow></mstyle></mtd><mtd><mstyle scriptlevel="0" displaystyle="false"><mrow><mtext>for&nbsp;</mtext><mi>i</mi><mo>=</mo><mn>0</mn></mrow></mstyle></mtd></mtr><mtr><mtd><mstyle scriptlevel="0" displaystyle="false"><mrow><mi>R</mi><mo>−</mo><mrow><mo fence="true">(</mo><mfrac><mrow><mi>L</mi><mo stretchy="false">[</mo><mn>0</mn><mo stretchy="false">]</mo></mrow><mrow><mi>v</mi><mi>a</mi><mi>r</mi><mi>s</mi><mo stretchy="false">[</mo><mn>0</mn><mo stretchy="false">]</mo></mrow></mfrac><mo>−</mo><mfrac><mrow><mi>L</mi><mo stretchy="false">[</mo><mn>0</mn><mo stretchy="false">]</mo></mrow><mrow><mi>A</mi><mo stretchy="false">[</mo><mn>0</mn><mo stretchy="false">]</mo></mrow></mfrac><mo fence="true">)</mo></mrow><mo>≥</mo><mn>0</mn></mrow></mstyle></mtd><mtd><mstyle scriptlevel="0" displaystyle="false"><mrow><mtext>for&nbsp;</mtext><mi>i</mi><mo>=</mo><mn>1</mn></mrow></mstyle></mtd></mtr><mtr><mtd><mstyle scriptlevel="0" displaystyle="false"><mtext>Subsequent&nbsp;constraints&nbsp;build&nbsp;upon&nbsp;the&nbsp;previous&nbsp;ones</mtext></mstyle></mtd><mtd><mstyle scriptlevel="0" displaystyle="false"><mrow><mtext>for&nbsp;</mtext><mi>i</mi><mo>≥</mo><mn>2</mn></mrow></mstyle></mtd></mtr></mtable></mrow></mrow><annotation encoding="application/x-tex">h_i(vars) = \left\{
\begin{array}{ll}
      R + \left( \frac{L[0]}{vars[0]} - \frac{L[0]}{A[0]} \right) \geq 0 &amp; \text{for } i=0 \\
      R - \left( \frac{L[0]}{vars[0]} - \frac{L[0]}{A[0]} \right) \geq 0 &amp; \text{for } i=1 \\
      \text{Subsequent constraints build upon the previous ones} &amp; \text{for } i \geq 2 \\
\end{array} 
\right.</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height:1em;vertical-align:-0.25em;"></span><span class="mord"><span class="mord mathnormal">h</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height:0.3117em;"><span style="top:-2.55em;margin-left:0em;margin-right:0.05em;"><span class="pstrut" style="height:2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathnormal mtight">i</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height:0.15em;"><span></span></span></span></span></span></span><span class="mopen">(</span><span class="mord mathnormal" style="margin-right:0.03588em;">v</span><span class="mord mathnormal">a</span><span class="mord mathnormal">rs</span><span class="mclose">)</span><span class="mspace" style="margin-right:0.2778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right:0.2778em;"></span></span><span class="base"><span class="strut" style="height:4.8em;vertical-align:-2.15em;"></span><span class="minner"><span class="mopen"><span class="delimsizing mult"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height:2.65em;"><span style="top:-1.9em;"><span class="pstrut" style="height:3.15em;"></span><span class="delimsizinginner delim-size4"><span>⎩</span></span></span><span style="top:-1.892em;"><span class="pstrut" style="height:3.15em;"></span><span style="height:0.616em;width:0.8889em;"><svg xmlns="http://www.w3.org/2000/svg" width="0.8889em" height="0.616em" style="width:0.8889em" viewBox="0 0 888.89 616" preserveAspectRatio="xMinYMin"><path d="M384 0 H504 V616 H384z M384 0 H504 V616 H384z"></path></svg></span></span><span style="top:-3.15em;"><span class="pstrut" style="height:3.15em;"></span><span class="delimsizinginner delim-size4"><span>⎨</span></span></span><span style="top:-4.292em;"><span class="pstrut" style="height:3.15em;"></span><span style="height:0.616em;width:0.8889em;"><svg xmlns="http://www.w3.org/2000/svg" width="0.8889em" height="0.616em" style="width:0.8889em" viewBox="0 0 888.89 616" preserveAspectRatio="xMinYMin"><path d="M384 0 H504 V616 H384z M384 0 H504 V616 H384z"></path></svg></span></span><span style="top:-4.9em;"><span class="pstrut" style="height:3.15em;"></span><span class="delimsizinginner delim-size4"><span>⎧</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height:2.15em;"><span></span></span></span></span></span></span><span class="mord"><span class="mtable"><span class="arraycolsep" style="width:0.5em;"></span><span class="col-align-l"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height:2.65em;"><span style="top:-4.65em;"><span class="pstrut" style="height:3.15em;"></span><span class="mord"><span class="mord mathnormal" style="margin-right:0.00773em;">R</span><span class="mspace" style="margin-right:0.2222em;"></span><span class="mbin">+</span><span class="mspace" style="margin-right:0.2222em;"></span><span class="minner"><span class="mopen delimcenter" style="top:0em;"><span class="delimsizing size2">(</span></span><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height:1.01em;"><span style="top:-2.655em;"><span class="pstrut" style="height:3em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathnormal mtight" style="margin-right:0.03588em;">v</span><span class="mord mathnormal mtight">a</span><span class="mord mathnormal mtight">rs</span><span class="mopen mtight">[</span><span class="mord mtight">0</span><span class="mclose mtight">]</span></span></span></span><span style="top:-3.23em;"><span class="pstrut" style="height:3em;"></span><span class="frac-line" style="border-bottom-width:0.04em;"></span></span><span style="top:-3.485em;"><span class="pstrut" style="height:3em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathnormal mtight">L</span><span class="mopen mtight">[</span><span class="mord mtight">0</span><span class="mclose mtight">]</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height:0.52em;"><span></span></span></span></span></span><span class="mclose nulldelimiter"></span></span><span class="mspace" style="margin-right:0.2222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right:0.2222em;"></span><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height:1.01em;"><span style="top:-2.655em;"><span class="pstrut" style="height:3em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathnormal mtight">A</span><span class="mopen mtight">[</span><span class="mord mtight">0</span><span class="mclose mtight">]</span></span></span></span><span style="top:-3.23em;"><span class="pstrut" style="height:3em;"></span><span class="frac-line" style="border-bottom-width:0.04em;"></span></span><span style="top:-3.485em;"><span class="pstrut" style="height:3em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathnormal mtight">L</span><span class="mopen mtight">[</span><span class="mord mtight">0</span><span class="mclose mtight">]</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height:0.52em;"><span></span></span></span></span></span><span class="mclose nulldelimiter"></span></span><span class="mclose delimcenter" style="top:0em;"><span class="delimsizing size2">)</span></span></span><span class="mspace" style="margin-right:0.2778em;"></span><span class="mrel">≥</span><span class="mspace" style="margin-right:0.2778em;"></span><span class="mord">0</span></span></span><span style="top:-2.85em;"><span class="pstrut" style="height:3.15em;"></span><span class="mord"><span class="mord mathnormal" style="margin-right:0.00773em;">R</span><span class="mspace" style="margin-right:0.2222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right:0.2222em;"></span><span class="minner"><span class="mopen delimcenter" style="top:0em;"><span class="delimsizing size2">(</span></span><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height:1.01em;"><span style="top:-2.655em;"><span class="pstrut" style="height:3em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathnormal mtight" style="margin-right:0.03588em;">v</span><span class="mord mathnormal mtight">a</span><span class="mord mathnormal mtight">rs</span><span class="mopen mtight">[</span><span class="mord mtight">0</span><span class="mclose mtight">]</span></span></span></span><span style="top:-3.23em;"><span class="pstrut" style="height:3em;"></span><span class="frac-line" style="border-bottom-width:0.04em;"></span></span><span style="top:-3.485em;"><span class="pstrut" style="height:3em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathnormal mtight">L</span><span class="mopen mtight">[</span><span class="mord mtight">0</span><span class="mclose mtight">]</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height:0.52em;"><span></span></span></span></span></span><span class="mclose nulldelimiter"></span></span><span class="mspace" style="margin-right:0.2222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right:0.2222em;"></span><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height:1.01em;"><span style="top:-2.655em;"><span class="pstrut" style="height:3em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathnormal mtight">A</span><span class="mopen mtight">[</span><span class="mord mtight">0</span><span class="mclose mtight">]</span></span></span></span><span style="top:-3.23em;"><span class="pstrut" style="height:3em;"></span><span class="frac-line" style="border-bottom-width:0.04em;"></span></span><span style="top:-3.485em;"><span class="pstrut" style="height:3em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathnormal mtight">L</span><span class="mopen mtight">[</span><span class="mord mtight">0</span><span class="mclose mtight">]</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height:0.52em;"><span></span></span></span></span></span><span class="mclose nulldelimiter"></span></span><span class="mclose delimcenter" style="top:0em;"><span class="delimsizing size2">)</span></span></span><span class="mspace" style="margin-right:0.2778em;"></span><span class="mrel">≥</span><span class="mspace" style="margin-right:0.2778em;"></span><span class="mord">0</span></span></span><span style="top:-1.36em;"><span class="pstrut" style="height:3.15em;"></span><span class="mord"><span class="mord text"><span class="mord">Subsequent&nbsp;constraints&nbsp;build&nbsp;upon&nbsp;the&nbsp;previous&nbsp;ones</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height:2.15em;"><span></span></span></span></span></span><span class="arraycolsep" style="width:0.5em;"></span><span class="arraycolsep" style="width:0.5em;"></span><span class="col-align-l"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height:2.65em;"><span style="top:-4.65em;"><span class="pstrut" style="height:3.15em;"></span><span class="mord"><span class="mord text"><span class="mord">for&nbsp;</span></span><span class="mord mathnormal">i</span><span class="mspace" style="margin-right:0.2778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right:0.2778em;"></span><span class="mord">0</span></span></span><span style="top:-2.85em;"><span class="pstrut" style="height:3.15em;"></span><span class="mord"><span class="mord text"><span class="mord">for&nbsp;</span></span><span class="mord mathnormal">i</span><span class="mspace" style="margin-right:0.2778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right:0.2778em;"></span><span class="mord">1</span></span></span><span style="top:-1.36em;"><span class="pstrut" style="height:3.15em;"></span><span class="mord"><span class="mord text"><span class="mord">for&nbsp;</span></span><span class="mord mathnormal">i</span><span class="mspace" style="margin-right:0.2778em;"></span><span class="mrel">≥</span><span class="mspace" style="margin-right:0.2778em;"></span><span class="mord">2</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height:2.15em;"><span></span></span></span></span></span><span class="arraycolsep" style="width:0.5em;"></span></span></span><span class="mclose nulldelimiter"></span></span></span></span></span></span></p>
<p>These constraints enforce a maximum allowable deviation (<code>R</code>) between the inverse of the variables <code>vars[i]</code> and the inverse of some reference values <code>A[i]</code>.</p>
<h5 id="smoothing">Smoothing </h5>
<p>Follow <a href="https://www.fon.hum.uva.nl/praat/manual/Intro_8_2__Manipulation_of_duration.html">Duration Manipulation with Praat Software</a></p>
<p>Praat Automatically do linear smoothening. So we have to just pass which position we should give what spped. Our algorithm decides that.</p>
<p>Mathematically, for a given speed-up factor <code>s</code> for a segment, setting a <code>DurationTier</code> point to <code>1/s</code> means that the segment will be played back at a rate <code>s</code> times its original speed. If s &gt; 1, the playback is faster; if s &lt; 1, the playback is slower. By adjusting these points gradually, the change in playback speed can be made smoother, so it's less jarring to listeners. The script mathematically calculates the positions of the <code>DurationTier</code> points to create a linear transition in playback speed between adjacent audio segments.</p>
<p>he position of the points in the <code>DurationTier</code> is calculated based on the relaxation value (<code>relaxation</code>), speed-up factors (<code>s1</code>, <code>s2</code>, <code>s3</code>), and the total duration of the original sound (<code>total_duration</code>). The relaxation value is used to determine a transition region where the speed changes gradually.</p>
<p>For the first sound file:</p>
<ul>
<li>We calculate <code>relaxation</code> based on the current and next duration.</li>
<li><code>r</code> is calculated as the minimum of <code>relaxation / s1</code> and <code>relaxation / s2</code>, ensuring that the transition region does not exceed the designated relaxation period and remains proportional to the speed-up factors.</li>
<li>Two points are then added to the <code>DurationTier</code> to define the start and end of the transition:
<ul>
<li>Start point: <code>call(duration_tier, "Add point", 0.00, 1/s1)</code></li>
<li>End of transition: <code>call(duration_tier, "Add point", total_duration - r * s1, 1/s1)</code></li>
</ul>
</li>
<li>A third point is added at the end of the audio (<code>total_duration</code>), at a speed (<code>1/speed</code>) that is an average of <code>s1</code> and <code>s2</code>, weighted by their relative difference:
<ul>
<li><code>speed = s1 + s1 * (s2 - s1) / (s1 + s2)</code></li>
</ul>
</li>
</ul>
<p>For the middle sound files, similar calculations are performed but with additional consideration for transitions at both the beginning and the end.</p>
<p>For the last sound file:</p>
<ul>
<li>The starting point for the <code>DurationTier</code> is set to an intermediary speed (<code>1/speed</code>) instead of starting at <code>1/s2</code>, and the transition end uses the same logic as above.</li>
</ul>
<p>Mathematically, we are defining a linear interpolation for the speed change over the transition region. Let's define:</p>
<ul>
<li><code>t0</code>: Start time of the transition region.</li>
<li><code>t1</code>: End time of the transition region within the total_duration.</li>
<li><code>s(t)</code>: Speed function over time.</li>
</ul>
<p>For the first sound file, we set <code>t0 = total_duration - r * s1</code> and <code>s(t0) = s1</code>. We want to find <code>s(t1)</code> at <code>t1 = total_duration</code>.</p>
<p>We can then define a linear interpolation of speed over time as follows:</p>
<p><span class="katex-display"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><semantics><mrow><mi>s</mi><mo stretchy="false">(</mo><mi>t</mi><mo stretchy="false">)</mo><mo>=</mo><mi>s</mi><mo stretchy="false">(</mo><mi>t</mi><mn>0</mn><mo stretchy="false">)</mo><mo>+</mo><mfrac><mrow><mo stretchy="false">(</mo><mi>s</mi><mo stretchy="false">(</mo><mi>t</mi><mn>1</mn><mo stretchy="false">)</mo><mo>−</mo><mi>s</mi><mo stretchy="false">(</mo><mi>t</mi><mn>0</mn><mo stretchy="false">)</mo><mo stretchy="false">)</mo></mrow><mrow><mo stretchy="false">(</mo><mi>t</mi><mn>1</mn><mo>−</mo><mi>t</mi><mn>0</mn><mo stretchy="false">)</mo></mrow></mfrac><mo>⋅</mo><mo stretchy="false">(</mo><mi>t</mi><mo>−</mo><mi>t</mi><mn>0</mn><mo stretchy="false">)</mo></mrow><annotation encoding="application/x-tex">s(t) = s(t0) + \frac{(s(t1) - s(t0))}{(t1 - t0)} \cdot (t - t0)</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height:1em;vertical-align:-0.25em;"></span><span class="mord mathnormal">s</span><span class="mopen">(</span><span class="mord mathnormal">t</span><span class="mclose">)</span><span class="mspace" style="margin-right:0.2778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right:0.2778em;"></span></span><span class="base"><span class="strut" style="height:1em;vertical-align:-0.25em;"></span><span class="mord mathnormal">s</span><span class="mopen">(</span><span class="mord mathnormal">t</span><span class="mord">0</span><span class="mclose">)</span><span class="mspace" style="margin-right:0.2222em;"></span><span class="mbin">+</span><span class="mspace" style="margin-right:0.2222em;"></span></span><span class="base"><span class="strut" style="height:2.363em;vertical-align:-0.936em;"></span><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height:1.427em;"><span style="top:-2.314em;"><span class="pstrut" style="height:3em;"></span><span class="mord"><span class="mopen">(</span><span class="mord mathnormal">t</span><span class="mord">1</span><span class="mspace" style="margin-right:0.2222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right:0.2222em;"></span><span class="mord mathnormal">t</span><span class="mord">0</span><span class="mclose">)</span></span></span><span style="top:-3.23em;"><span class="pstrut" style="height:3em;"></span><span class="frac-line" style="border-bottom-width:0.04em;"></span></span><span style="top:-3.677em;"><span class="pstrut" style="height:3em;"></span><span class="mord"><span class="mopen">(</span><span class="mord mathnormal">s</span><span class="mopen">(</span><span class="mord mathnormal">t</span><span class="mord">1</span><span class="mclose">)</span><span class="mspace" style="margin-right:0.2222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right:0.2222em;"></span><span class="mord mathnormal">s</span><span class="mopen">(</span><span class="mord mathnormal">t</span><span class="mord">0</span><span class="mclose">))</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height:0.936em;"><span></span></span></span></span></span><span class="mclose nulldelimiter"></span></span><span class="mspace" style="margin-right:0.2222em;"></span><span class="mbin">⋅</span><span class="mspace" style="margin-right:0.2222em;"></span></span><span class="base"><span class="strut" style="height:1em;vertical-align:-0.25em;"></span><span class="mopen">(</span><span class="mord mathnormal">t</span><span class="mspace" style="margin-right:0.2222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right:0.2222em;"></span></span><span class="base"><span class="strut" style="height:1em;vertical-align:-0.25em;"></span><span class="mord mathnormal">t</span><span class="mord">0</span><span class="mclose">)</span></span></span></span></span></p>
<p>Where <span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>t</mi></mrow><annotation encoding="application/x-tex">t</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height:0.6151em;"></span><span class="mord mathnormal">t</span></span></span></span> falls within <span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>t</mi><mn>0</mn></mrow><annotation encoding="application/x-tex">t0</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height:0.6444em;"></span><span class="mord mathnormal">t</span><span class="mord">0</span></span></span></span> and <span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>t</mi><mn>1</mn></mrow><annotation encoding="application/x-tex">t1</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height:0.6444em;"></span><span class="mord mathnormal">t</span><span class="mord">1</span></span></span></span>, which defines a straight line between the points <span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mo stretchy="false">(</mo><mi>t</mi><mn>0</mn><mo separator="true">,</mo><mi>s</mi><mo stretchy="false">(</mo><mi>t</mi><mn>0</mn><mo stretchy="false">)</mo><mo stretchy="false">)</mo></mrow><annotation encoding="application/x-tex">(t0, s(t0))</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height:1em;vertical-align:-0.25em;"></span><span class="mopen">(</span><span class="mord mathnormal">t</span><span class="mord">0</span><span class="mpunct">,</span><span class="mspace" style="margin-right:0.1667em;"></span><span class="mord mathnormal">s</span><span class="mopen">(</span><span class="mord mathnormal">t</span><span class="mord">0</span><span class="mclose">))</span></span></span></span> and <span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mo stretchy="false">(</mo><mi>t</mi><mn>1</mn><mo separator="true">,</mo><mi>s</mi><mo stretchy="false">(</mo><mi>t</mi><mn>1</mn><mo stretchy="false">)</mo><mo stretchy="false">)</mo></mrow><annotation encoding="application/x-tex">(t1, s(t1))</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height:1em;vertical-align:-0.25em;"></span><span class="mopen">(</span><span class="mord mathnormal">t</span><span class="mord">1</span><span class="mpunct">,</span><span class="mspace" style="margin-right:0.1667em;"></span><span class="mord mathnormal">s</span><span class="mopen">(</span><span class="mord mathnormal">t</span><span class="mord">1</span><span class="mclose">))</span></span></span></span> on a graph of speed over time.</p>
<p>By setting points at <span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>t</mi><mn>0</mn></mrow><annotation encoding="application/x-tex">t0</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height:0.6444em;"></span><span class="mord mathnormal">t</span><span class="mord">0</span></span></span></span> and <span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>t</mi><mn>1</mn></mrow><annotation encoding="application/x-tex">t1</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height:0.6444em;"></span><span class="mord mathnormal">t</span><span class="mord">1</span></span></span></span>, we're effectively dictating the slope of our line, creating a gradual transition from one speed to another over the duration of the relaxation period, avoiding sudden changes in playback speed.</p>
<h1>Audio Outputs</h1>
<h2 id="issue-3">Issue 3 </h2>
<ul>
<li>Audio Qaulity Seems to be dropped</li>
<li>Tested with no smootening</li>
<li>Using Ffmpeg for speed up</li>
</ul>
<p>Samples</p>
<table>
<thead>
<tr>
<th></th>
<th>With Smoothening</th>
<th>Without Smoothening</th>
</tr>
</thead>
<tbody>
<tr>
<td>Id (1656)</td>
<td><a href="https://drive.google.com/file/d/1fvgfjklSiEBLWC7i_0eVHyX50NOWS4TM/view?usp=drive_link">Video</a></td>
<td><a href="https://drive.google.com/file/d/1yRtvRwiO-MV1JyFRVw2pMeEyFH5T2ey6/view?usp=drive_link">Video</a></td>
</tr>
</tbody>
</table>
<h2 id="all-algo-comparasion">All Algo Comparasion </h2>
<p>Samples</p>
<table>
<thead>
<tr>
<th></th>
<th>Semantic Shortening</th>
<th>Our New Algo</th>
<th>Our New Algo + Semantic Shortening</th>
<th>ILP</th>
<th>ILP + Semantic Shortening</th>
</tr>
</thead>
<tbody>
<tr>
<td>Id (1695)</td>
<td><a href="https://drive.google.com/file/d/1yqQROH4AZ_CiZ5nEv7Gdfmqickw0IQEC/view?usp=drive_link">Video</a></td>
<td><a href="https://drive.google.com/file/d/1jBhT8an1ywZ9ednmTpbWysdR5jfJQvG7/view?usp=drive_link">Video</a></td>
<td><a href="https://drive.google.com/file/d/1xHw7R9dmMSJH2K7CsCfNMq1UZge4ocqJ/view?usp=drive_link">Video</a></td>
<td></td>
<td></td>
</tr>
</tbody>
</table>
<h2 id="issue-2">Issue 2 </h2>
<ul>
<li>Play back rate issue: In previous algorthim we dont handle f we have less than 1x playback rate</li>
<li>Handled this in preprocessing step (Add silence if output audio length is less than src audio)</li>
<li>Change the Optmization Algorithm to have constraints on play back rate &lt; 1</li>
</ul>
<p>Samples</p>
<table>
<thead>
<tr>
<th></th>
<th>Previous Algo</th>
<th>Fixed Algo</th>
<th>TimeStamps</th>
</tr>
</thead>
<tbody>
<tr>
<td>Id (1398)</td>
<td><a href="https://drive.google.com/file/d/1lCzG5349LYWengwvaMb0PxRzZpOckmR2/view?usp=drive_link">Audio</a></td>
<td><a href="https://drive.google.com/file/d/16szKWzuMN2bH023Y6XH1ErUMer4Os-Wf/view?usp=drive_link">Audio</a></td>
<td>0:31 - 0:35</td>
</tr>
</tbody>
</table>
<h2 id="issue-1">Issue 1 </h2>
<ul>
<li>Cut off Issue: Sometimes there is  10ms to 1sec cut off in dialgoue due to precision</li>
<li>Handle this in post-processing step</li>
<li>if cut off is there then we speed up the dialogue little bit more to fit inside that dialogue frame</li>
</ul>
<p>Samples</p>
<table>
<thead>
<tr>
<th></th>
<th>Baseline</th>
<th>Previous Algo</th>
<th>Fixed Algo</th>
<th>TimeStamps</th>
</tr>
</thead>
<tbody>
<tr>
<td>Id (1411)</td>
<td><a href="https://drive.google.com/file/d/1ad0BH9tUaoviH9-4_FpmFd4xRTAurJPK/view?usp=drive_link">Audio</a></td>
<td><a href="https://drive.google.com/file/d/1lzQXFa4YbpCKeessBZ3tCOkIPNQQoqOw/view?usp=drive_link">Audio</a></td>
<td><a href="https://drive.google.com/file/d/1PxNuvRUM_Ht6j9uE1Zn8uGEPBLYtTMZk/view?usp=drive_link">Audio</a></td>
<td>2:04 - 2:15</td>
</tr>
</tbody>
</table>
<h2 id="approach-3">Approach 3 </h2>
<ul>
<li>Audio Smoothenting + Clsuter Based</li>
<li>In this we assume for a particlular cluster start and end time will be preserved</li>
</ul>
<table>
<thead>
<tr>
<th></th>
<th>Baseline</th>
<th>Clustering + Realxation</th>
<th>Clustering + Smoothening</th>
<th>Clustering + Realxation + Smoothening</th>
</tr>
</thead>
<tbody>
<tr>
<td>Id(4410)</td>
<td><a href="../../../assets/audios/baseline_audio_approch3.wav">Audio</a></td>
<td><a href="../../../assets/audios/optimal_speeds_with_naive_cluster_and_relaxation_0_5.wav">Audio</a></td>
<td><a href="../../../assets/audios/audio_smoothening.wav">Audio</a></td>
<td><a href="../../../assets/audios/optimal_speeds_with_naive_cluster_and_relaxation_0_5-wsmoothening.wav">Audio</a></td>
</tr>
<tr>
<td>Id(4514)_id</td>
<td><a href="https://drive.google.com/file/d/1je0R4Aax_A5BgIcpEqYCO-cZe_4Mav9e/view?usp=drive_link">Audio</a> , <a href="https://drive.google.com/file/d/1Jw0G5VwJnEFzqR2nw93R8DTnSAqrYiIO/view?usp=drive_link">Video</a></td>
<td><a href="https://drive.google.com/file/d/1WA6rOQXkjlCyDySzeeKyLDwckoBYMwAo/view?usp=drive_link">Audio</a></td>
<td><a href="https://drive.google.com/file/d/14XLkRCjkVoaXS5HIlT-24rc5OmYRuUdx/view?usp=drive_link">Audio</a></td>
<td><a href="https://drive.google.com/file/d/1YgGtPasSVbo520Vb6bv22_sc32_RUxM_/view?usp=drive_link">Audio</a> , <a href="https://drive.google.com/file/d/1i8Ma3TU1tDKMp9UlHsKC-2MfhTgYyW6w/view?usp=drive_link">Video</a></td>
</tr>
<tr>
<td>Id(4514)_hi</td>
<td><a href="https://drive.google.com/file/d/15Hp0qonLPCvlmwIUYXXLz5CXgD1dlB69/view?usp=drive_link">Audio</a></td>
<td><a href="https://drive.google.com/file/d/1tcj52nrnJKZLeTAk2Be2x0ffIZfBQCpq/view?usp=drive_link">Audio</a></td>
<td><a href="https://drive.google.com/file/d/14XLkRCjkVoaXS5HIlT-24rc5OmYRuUdx/view?usp=drive_link">Audio</a></td>
<td><a href="https://drive.google.com/file/d/1OsOK6YI1N57AR6AAkv9ejvdZ2rhrWWOh/view?usp=drive_link">Audio</a></td>
</tr>
</tbody>
</table>
<p><img src="../../../assets/images/plot.png" alt="SpeedUpOptimisation"></p>
<h6 id="baseline">Baseline </h6>
<p><audio controls="" controlslist="nodownload noplaybackrate" class="audio-smaller"><source src="/assets/audios/baseline_audio_approch3.wav"></audio></p>
<h6 id="clustering--realxation--smoothening">Clustering + Realxation + Smoothening </h6>
<p><audio controls="" controlslist="nodownload noplaybackrate" class="audio-smaller"><source src="/assets/audios/optimal_speeds_with_naive_cluster_and_relaxation_0_5-wsmoothening.wav"></audio></p>
<h6 id="clustering--realxation">Clustering + Realxation </h6>
<p><audio controls="" controlslist="nodownload noplaybackrate" class="audio-smaller"><source src="/assets/audios/optimal_speeds_with_naive_cluster_and_relaxation_0_5.wav"></audio></p>
<h6 id="clustering--smoothening">Clustering + Smoothening </h6>
<p><audio controls="" controlslist="nodownload noplaybackrate" class="audio-smaller"><source src="/assets/audios/audio_smoothening.wav"></audio></p>
<h2 id="approach-2">Approach 2 </h2>
<ul>
<li>audio smoothening + Video Smoothening</li>
<li>In this we assume every dailgoue's start and end time will be preserved</li>
<li>Here Gap paramter mean if less than this we will add this to previous cluster segment</li>
</ul>
<h6 id="baseline-1">Baseline </h6>
<video controls="" controlslist="nodownload" class="video-smaller">
  <source src="/assets/videos/out_baseline.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>
<h6 id="only-audio-smoothening">Only Audio Smoothening </h6>
<video controls="" controlslist="nodownload" class="video-smaller">
  <source src="/assets/videos/out_audio.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>
<h6 id="video-smooth--audio-smooth-gap-paramter-12">Video Smooth + Audio Smooth (Gap Paramter 1.2) </h6>
<video controls="" controlslist="nodownload" class="video-smaller">
  <source src="/assets/videos/out_video.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>
<h6 id="video-smooth--audio-smooth-gap-paramter-3">Video Smooth + Audio Smooth (Gap Paramter 3) </h6>
<video controls="" controlslist="nodownload" class="video-smaller">
  <source src="/assets/videos/output_video_2.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>
<h2 id="approach-1">Approach 1 </h2>
<ul>
<li>Only audio smoothening</li>
<li>In this we assume every dailgoue's start and end time will be preserved</li>
<li>Gap paramter(G) mean if less than this we will add this to previous cluster segment</li>
<li>Relaxation paramter(R) mean when smootehnng we take this much relaxation</li>
</ul>
<h4 id="baseline-2">Baseline </h4>
<p><a href="../../../assets/audios/runid_baseline_21sec.wav">Audio</a></p>
<table>
<thead>
<tr>
<th></th>
<th>R(0.1)</th>
<th>R(0.2)</th>
<th>R(0.4)</th>
<th>R(0.5)</th>
<th>R(0.7)</th>
<th>R(0.9)</th>
<th>R(1.0)</th>
<th>R(1.2)</th>
<th>R(1.5)</th>
</tr>
</thead>
<tbody>
<tr>
<td>G(0.5)</td>
<td><a href="../../../assets/audios/final_runid_21sec__0.5_0.1.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__0.5_0.2.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__0.5_0.4.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__0.5_0.5.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__0.5_0.7.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__0.5_0.9.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__0.5_1.0.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__0.5_1.2.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__0.5_1.5.wav">Audio</a></td>
</tr>
<tr>
<td>G(0.8)</td>
<td><a href="../../../assets/audios/final_runid_21sec__0.8_0.1.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__0.8_0.2.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__0.8_0.4.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__0.8_0.5.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__0.8_0.7.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__0.8_0.9.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__0.8_1.0.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__0.8_1.2.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__0.8_1.5.wav">Audio</a></td>
</tr>
<tr>
<td>G(1.0)</td>
<td><a href="../../../assets/audios/final_runid_21sec__1.0_0.1.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.0_0.2.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.0_0.4.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.0_0.5.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.0_0.7.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.0_0.9.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.0_1.0.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.0_1.2.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.0_1.5.wav">Audio</a></td>
</tr>
<tr>
<td>G(1.2)</td>
<td><a href="../../../assets/audios/final_runid_21sec__1.2_0.1.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.2_0.2.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.2_0.4.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.2_0.5.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.2_0.7.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.2_0.9.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.2_1.0.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.2_1.2.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.2_1.5.wav">Audio</a></td>
</tr>
<tr>
<td>G(1.5)</td>
<td><a href="../../../assets/audios/final_runid_21sec__1.5_0.1.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.5_0.2.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.5_0.4.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.5_0.5.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.5_0.7.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.5_0.9.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.5_1.0.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.5_1.2.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.5_1.5.wav">Audio</a></td>
</tr>
<tr>
<td>G(1.8)</td>
<td><a href="../../../assets/audios/final_runid_21sec__1.8_0.1.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.8_0.2.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.8_0.4.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.8_0.5.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.8_0.7.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.8_0.9.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.8_1.0.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.8_1.2.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__1.8_1.5.wav">Audio</a></td>
</tr>
<tr>
<td>G(2.0)</td>
<td><a href="../../../assets/audios/final_runid_21sec__2.0_0.1.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__2.0_0.2.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__2.0_0.4.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__2.0_0.5.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__2.0_0.7.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__2.0_0.9.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__2.0_1.0.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__2.0_1.2.wav">Audio</a></td>
<td><a href="../../../assets/audios/final_runid_21sec__2.0_1.5.wav">Audio</a></td>
</tr>
</tbody>
</table>
